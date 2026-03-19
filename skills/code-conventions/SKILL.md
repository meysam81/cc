---
name: code-conventions
description: >
  Enforces personal code conventions for Go, TypeScript/Vue, and Astro projects.
  Covers error handling, HTTP patterns, design tokens, DRY principles, struct usage,
  functional options, and tooling. Auto-triggers on Go, TypeScript, Vue, and Astro files.
  Libraries are preferred defaults — use them unless the project has a reason not to.
globs:
  - "**/*.go"
  - "**/*.ts"
  - "**/*.vue"
  - "**/*.astro"
---

# Code Conventions

Opinionated coding conventions for Go, TypeScript/Vue, and Astro projects.
Every rule exists because it prevented a real bug or eliminated real friction.

Priority: **Clarity > Simplicity > Concision > Maintainability > DRY**.

---

## Part 1: Go Conventions

### 1.1 CLI Framework

Preferred: `urfave/cli/v3`.

- Use `Destination: &var` + `Sources: cli.EnvVars("ENV_NAME")` for every flag.
- One subcommand per package under `cmd/<name>/`.
- Entry point in `main.go` calls `app.Run()`.

```go
// Good.
&cli.BoolFlag{
    Name:        "metrics",
    Usage:       "Enable Prometheus metrics",
    Value:       true,
    Sources:     cli.EnvVars("APP_METRICS"),
    Destination: &metricsFlag,
}

// Bad: positional args, no env var source.
app.Action = func(c *cli.Context) error {
    host := c.Args().First()
}
```

### 1.2 Logger

Preferred: `github.com/meysam81/x/logging` thin wrapper around zerolog.

- Project wraps in `internal/logger/` with `type Logger = zerolog.Logger`.
- ALL packages receive `*logger.Logger` via constructor injection.
- NEVER import `github.com/rs/zerolog/log` (the global logger). Only `internal/logger/` may.
- Enforce with `depguard` linter rule.

```go
// Good: DI via constructor.
func NewService(store Store, log *logger.Logger) *Service {
    return &Service{store: store, log: log}
}

// Bad: global logger import.
import "github.com/rs/zerolog/log"
func doWork() { log.Info().Msg("bad") }
```

### 1.3 Config / Environment

Preferred: `caarlos0/env/v11`.

- Struct tags: `env:"VAR_NAME" envDefault:"value"`.
- `Validate()` returns `[]string` of ALL errors (accumulate, don't fail on first).
- Never use `required` tags — validate manually for better error messages.

```go
type Config struct {
    Port          int    `env:"PORT" envDefault:"8080"`
    EncryptionKey string `env:"ENCRYPTION_KEY"`
}

func (c *Config) Validate() (errs []string) {
    if c.EncryptionKey == "" {
        errs = append(errs, "ENCRYPTION_KEY is required")
    }
    if c.Port < 1 || c.Port > 65535 {
        errs = append(errs, "PORT must be 1-65535")
    }
    return errs
}
```

### 1.4 HTTP Framework

Preferred: chi via `github.com/meysam81/x/chimux`.

- `chimux.NewChi()` with functional options: `WithLoggingMiddleware()`, `WithMetrics()`, `WithHealthz()`, `WithLogger()`.
- Built-in: CleanPath, RealIP, Recoverer (disable with `WithDisable*` options).
- Structured logging middleware masks sensitive headers automatically.
- Use `r.With()` for per-route middleware. Never check HTTP methods inside handlers.

```go
r := chimux.NewChi(
    chimux.WithLoggingMiddleware(),
    chimux.WithLogger(log),
    chimux.WithMetrics(),
    chimux.WithHealthz(),
)
r.With(authMW).Get("/api/users", handleUsers)
```

### 1.5 Error Handling

**NEVER assign errors to `_`.** Every error must be logged, returned, or handled.
This includes `rows.Close()`, `json.Marshal()`, `result.RowsAffected()`, ALL stdlib/lib calls.

```go
// Good.
if err := store.Close(); err != nil {
    log.Error().Err(err).Msg("close store")
}

// Bad.
_ = store.Close()
```

- Wrap errors with context: `fmt.Errorf("load user %s: %w", id, err)`.
- NEVER expose `err.Error()` in API responses. Log raw errors internally, return user-friendly messages.
- Use an `httperr` package with constants for all user-facing messages.

```go
// Good: user-friendly error, raw error logged internally.
httperr.WriteError(w, http.StatusNotFound, httperr.DomainNotFound)

// Bad: leaks internal error details.
http.Error(w, err.Error(), 500)
```

### 1.6 JSON

Preferred: `github.com/goccy/go-json`. Never stdlib `encoding/json`.

- `writeJSON` helper on the Server struct.
- Use `json.NewEncoder(w).Encode()` for responses.

```go
import "github.com/goccy/go-json"

func (s *Server) writeJSON(w http.ResponseWriter, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    if err := json.NewEncoder(w).Encode(data); err != nil {
        s.log.Error().Err(err).Msg("failed to encode JSON")
    }
}
```

### 1.7 Storage / SQL

Preferred: sqlc for query generation, go-migrate for schema migrations.

- Use `sqlc.arg(name)` named params. Positional `?` only for 1-2 simple args.
- NEVER use `RETURNING *` — explicit column lists only.
- Queries in `internal/storage/queries/*.sql`, generated code in `internal/db/` (NEVER edit generated code).
- Migrations in `internal/storage/migrations/XXXXXX_<name>.{up,down}.sql`.

```sql
-- Good.
-- name: GetUser :one
SELECT id, email, name FROM users WHERE id = sqlc.arg(user_id);

-- Bad: RETURNING *, positional params.
INSERT INTO users (email) VALUES (?) RETURNING *;
```

### 1.8 SQLite

Preferred: `github.com/meysam81/x/sqlite`.

- Build-tag driver split: `mattn/go-sqlite3` (CGO), `modernc.org/sqlite` (pure-Go fallback).
- WAL journal mode by default. `MaxOpenConns=1` for SQLite.

```go
db, err := sqlite.NewDB(ctx, "app.db",
    sqlite.WithJournalMode("wal"),
    sqlite.WithConnMaxOpen(1),
)
```

### 1.9 Primary Keys

Preferred: TSID (Time-Sorted IDs).

- Internal: `int64`. External (API/JWT/URLs): Crockford Base32 string.
- `JSONID` type auto-marshals int64 to/from Base32 in JSON.
- `pathID(r, "name")` to decode URL params. `tsid.Encode(id)` in tests.

```go
type UserResponse struct {
    ID   tsid.JSONID `json:"id"`   // marshals as "01ARZ3NDEKTSV4RRFFQ69G5FAV"
    Name string      `json:"name"`
}
```

### 1.10 Structs

- NEVER use inline/anonymous structs. Define named types.
- NEVER use `map[string]interface{}` or `map[string]any`. Define proper structs.
- Named types for ALL request/response bodies.
- `internal/models/` for cross-package shared types.

```go
// Good.
type CreateUserRequest struct {
    Email string `json:"email"`
    Name  string `json:"name"`
}

// Bad: inline struct.
json.NewDecoder(r.Body).Decode(&struct {
    Email string `json:"email"`
}{})

// Bad: bare map.
data := map[string]interface{}{"email": email}
```

### 1.11 Interfaces

- Define narrow interfaces at the **call site**, not where implemented.
- Accept interfaces, return concrete types.
- Use function types for simple callbacks.

```go
// Good: narrow interface at call site.
type Store interface {
    LogAudit(ctx context.Context, entry AuditEntry) error
}

// Bad: fat interface defined alongside implementation.
type UserRepository interface {
    Create(...) error
    Read(...) error
    Update(...) error
    Delete(...) error
    List(...) error
    Search(...) error
    // 20 more methods...
}
```

### 1.12 Enums

- Go `const` with custom string types. NEVER use DB ENUMs.
- Database stores plain strings. Application enforces valid values.

```go
type Role string

const (
    RoleOwner  Role = "owner"
    RoleAdmin  Role = "admin"
    RoleViewer Role = "viewer"
)
```

### 1.13 Goroutines

- No sporadic `go func()`. Use structured concurrency.
- Preferred: `sync.WaitGroup` with `wg.Go(func(){})` or a job pool.
- Context-aware tickers: always have a `<-ctx.Done()` select leg.

```go
// Good: structured, context-aware.
wg.Go(func() {
    ticker := time.NewTicker(interval)
    defer ticker.Stop()
    for {
        select {
        case <-ticker.C:
            doWork()
        case <-ctx.Done():
            return
        }
    }
})

// Bad: fire-and-forget.
go func() {
    for range time.Tick(interval) {
        doWork()
    }
}()
```

### 1.14 HTTP Server Hardening

Always set on `http.Server`:

- `ReadHeaderTimeout` (prevent slowloris)
- `MaxHeaderBytes`
- Body size limits via `http.MaxBytesReader`
- Security headers: HSTS, CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy

### 1.15 HTTP Client

- DRY: centralize in `internal/httpclient/`.
- ALWAYS use `http.NewRequestWithContext`. Never bare `http.NewRequest`.
- `context.WithTimeout` for bounded operations.
- Retry logic with exponential backoff.

```go
// Good.
ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
defer cancel()
req, err := http.NewRequestWithContext(ctx, "GET", url, nil)

// Bad.
req, _ := http.NewRequest("GET", url, nil)
```

### 1.16 Crypto

Preferred: `github.com/meysam81/x/cryptox`.

- `Cipher` interface with `Encrypt(plaintext) / Decrypt(ciphertext)`.
- AES-256-GCM (default), ChaCha20-Poly1305, Argon2id / PBKDF2 password-based.
- `GenerateKey256()` for secure key generation. Always `crypto/rand`.

### 1.17 SMTP

Preferred: `github.com/meysam81/x/smtpclient`.

- Functional options: `WithTimeout()`, `WithRetry()`, `WithAuthType()`.
- TLS/STARTTLS support. Multipart messages with attachments.
- Retry with exponential backoff. Context-aware sending.

### 1.18 Config Structs & Functional Options

- If `New()` takes >2 args, use a `Config` struct. Validate inside `New()`.
- Functional options pattern: `type Option func(*options)`, `With*()` constructors.
- Always provide sensible defaults in the constructor.

```go
type Option func(*options)

type options struct {
    timeout time.Duration
    retries int
}

func WithTimeout(d time.Duration) Option {
    return func(o *options) { o.timeout = d }
}

func New(opts ...Option) *Client {
    o := &options{timeout: 30 * time.Second, retries: 3} // sensible defaults
    for _, opt := range opts {
        opt(o)
    }
    return &Client{opts: o}
}
```

### 1.19 Context

- Always accept `ctx context.Context` as first parameter.
- `context.WithTimeout` for bounded operations.
- `context.WithValue` for request-scoped data with typesafe keys (unexported key type).

### 1.20 Time

- Default format: `time.RFC3339`.
- Config intervals as seconds (int). Code-level durations as `time.Duration`.

### 1.21 Control Flow

- `switch` over `if/else` chains when >2 branches. Enforced by `gocritic ifElseChain`.
- Happy path flows straight down. Handle errors immediately with early returns.

```go
// Good.
switch {
case status >= 500:
    event = log.Error()
case status >= 400:
    event = log.Warn()
default:
    event = log.Info()
}

// Bad.
if status >= 500 {
    event = log.Error()
} else if status >= 400 {
    event = log.Warn()
} else {
    event = log.Info()
}
```

### 1.22 Testing

- Table-driven tests with `tt` loop variable.
- Narrow mock interfaces (test-specific, not shared).
- `httptest.NewRequest` + `httptest.NewRecorder` for handler tests.
- `synctest.Test` for goroutine tests. `t.Helper()` on helper functions.

### 1.23 Structured Logging

- Zerolog structured fields: `.Str()`, `.Int()`, `.Err()`, `.Msg()`.
- Always include context fields: `user_id`, `org_id`, `ip` where available.
- Log durations for operations. Warn for slow queries.
- Log level by status: 5xx → Error, 4xx → Warn, 2xx/3xx → Info.

### 1.24 DRY

- **Grep before creating utilities.** Check if the function already exists.
- Extract shared code to `internal/<pkg>/` with a focused responsibility.
- Anti-example: `extractIP` duplicated across `audit/middleware.go` and `ratelimit/ratelimit.go`.

### 1.25 Package Naming

- One responsibility per package. Never `util`, `helper`, `common`, `model`.
- Package name describes what it provides, not what it contains.
- Lowercase only, no underscores, no mixedCaps.

### 1.26 API Routes

- All user-facing endpoints under `/api/`.
- Internal endpoints at their own root (`/metrics`, `/healthz`).
- Enables clean network policy routing.

### 1.27 Email Templates

- MJML source templates + `go:generate bunx mjml` to compile to HTML.
- Compiled HTML embedded via `//go:embed`.
- Production email headers: List-Unsubscribe, postal address in footer.

### 1.28 Graceful Shutdown

- `signal.NotifyContext(ctx, os.Interrupt, syscall.SIGTERM)`.
- Soft timeout → hard timeout pattern.
- Drain buffered channels before exit. Close DB connections.

### 1.29 Observability

Preferred: `github.com/meysam81/x/tracing` for OpenTelemetry.

- OTLP HTTP export, chi-compatible middleware.
- Auto-shutdown on context cancellation.
- Prometheus metrics via chimux `WithMetrics()`.

### 1.30 CSRF

- `X-CSRF-Token` header. Middleware enforcement on mutations.
- Token generated server-side, returned in cookie or response body.

### 1.31 Single Binary

- `//go:embed dist` for SPA static assets.
- `//go:generate sqlc generate` for SQL.
- `//go:generate bunx mjml ...` for email templates.
- Single binary deployment to scratch container.

---

## Part 2: TypeScript / Vue Conventions

### 2.1 Build

- Preferred: Vite.
- Production: minify (esbuild), `cssMinify` (lightningcss), Brotli compression plugin.
- Manual chunks for vendor splitting. `assetsInlineLimit: 4096`.

### 2.2 HTTP Client

- Centralized `ky` wrapper in `lib/api.{js,ts}`. NEVER use raw `fetch()`.
- NEVER `import ky from "ky"` directly — always through the centralized module.
- Auth hooks, CSRF injection, org-id header injection built into the wrapper.
- Enforce with ESLint `no-restricted-imports` rule.

```typescript
// Good: centralized wrapper.
import { api } from "@/lib/api";
const user = await api.get("users/me").json();

// Bad: raw fetch.
const res = await fetch("/api/users/me");

// Bad: direct ky import.
import ky from "ky";
```

### 2.3 Validation

- Preferred: Zod for schemas.
- Shared validators in `shared/validation.ts` — zero-dependency pure functions.
- Centralized `VALIDATION_MSG` object for user-facing messages.

### 2.4 Design Tokens

**3-layer CSS variable system** in `shared/design-tokens.css`:

- **L1 Primitives**: raw color values (teal, slate, green, amber, red, blue, etc.)
- **L2 Semantic**: purpose-driven aliases (brand, bg, text, borders, status)
- **L3 Component**: specific component tokens (buttons, cards, nav, badges)

NEVER hardcode `#hex`, `rgb()`, or `rgba()` in `.vue` or `.astro` files.
Both light and dark themes defined in the same token file.

Exceptions: SVG inline attrs, `getComputedStyle()` fallbacks, color picker inputs.

```css
/* Good: use semantic token. */
.card {
  background: var(--color-bg-card);
}

/* Bad: hardcoded color. */
.card {
  background: #ffffff;
}
```

### 2.5 Layout

- `.page-container` (1200px), `--md` (1024px), `--sm` (768px) from shared CSS.
- NEVER set `max-width` or `margin: 0 auto` in scoped styles.
- Page headers: `.page-header` / `.page-title` / `.page-subtitle` globals.

### 2.6 Vue Components

- `<script setup>` always. Composition API only.
- `defineProps<{}>()` and `defineEmits<{}>()` with TypeScript generics.
- Named slots for flexibility. `v-if` for conditional rendering.

### 2.7 State Management

- Preferred: Pinia with Composition API.
- `defineStore("name", () => {})` with explicit return object.
- `ref()` for state, `computed()` for derived values.

```typescript
export const useAuthStore = defineStore("auth", () => {
  const user = ref<User | null>(null);
  const isLoggedIn = computed(() => !!user.value);
  function logout() {
    user.value = null;
  }
  return { user, isLoggedIn, logout };
});
```

### 2.8 Data Fetching

- Preferred: TanStack Vue Query.
- Reactive `queryKey` via `computed()`. Set `staleTime` for caching.
- Mutation invalidation to keep data fresh. One composable per domain.

### 2.9 TypeScript Migration

- Boy scout rule: new files in `.ts`, gradually migrate `.js`.
- Shared modules (`shared/`) always TypeScript.
- Use `defineProps<{}>()` generics, not `defineProps({})` options syntax.

### 2.10 Composables

- `use*` prefix. Return reactive refs + functions.
- Pattern: validate → set loading → pushUrl → run async function.

```typescript
export function useDomainTool(opts: ToolOptions) {
  const domain = ref("");
  const loading = ref(false);
  const result = ref(null);

  function validate() {
    return !!domain.value.trim();
  }

  async function run() {
    if (!validate()) return;
    loading.value = true;
    try {
      result.value = await opts.fetcher(domain.value);
      opts.pushUrl?.(domain.value);
    } finally {
      loading.value = false;
    }
  }

  return { domain, loading, result, validate, run };
}
```

### 2.11 Router

- Lazy loading: `() => import('./views/UserView.vue')`.
- Route meta for title, breadcrumb, required permissions.
- Auth guards. Safe redirect validation (no open redirects).

### 2.12 CSS

- Scoped styles. CSS custom properties only (from design tokens). No CSS frameworks.
- Responsive via media queries + `clamp()` for fluid typography.
- Mobile-first approach.

### 2.13 Shared Data

- `shared/site.ts` + `shared/pricing.ts` for DRY constants.
- Never hardcode protocol counts, pricing, feature limits in components.

### 2.14 ESLint

- Vue flat config with recommended rules.
- ky enforcement rules (no-restricted-imports for `fetch` and direct `ky`).
- Prettier compatibility overrides.

### 2.15 Package Management

- `bun` always. Never `npm`.
- Script name: `"start"` (never `"dev"`).
- `"type": "module"` in `package.json`.

### 2.16 Error States

- `try/catch` with user-friendly messages. Never expose internal details.
- Pattern: `v-if="errorMsg"` for conditional error display.

### 2.17 Forms

- `v-model` + explicit validation functions.
- `@submit.prevent`. Loading state during submission.

---

## Part 3: Astro Conventions

### 3.1 Framework Setup

- Astro with SSG (`output: "static"`).
- Content collections for structured data (blog, learn, research).
- MDX for rich content pages. Vue for interactive islands.

### 3.2 Sitemap

- `@astrojs/sitemap` integration with XSL stylesheet via `xslUrl`.
- Produces styled, human-readable XML sitemaps.

### 3.3 RSS

- Styled RSS feed with XSL stylesheet.
- `rss.xml.ts` endpoint with proper `<channel>` metadata.

### 3.4 SEO & Machine-Readable Files

- `robots.txt` as static Astro page.
- `llms.txt` — concise site description for LLM crawlers.
- `llms-full.txt.ts` — comprehensive version generated from content collections.

### 3.5 Search

- Pagefind for client-side static search (zero JS bundle cost at build time).
- Custom CSS overrides in `pagefind-overrides.css` for brand consistency.

### 3.6 Compression

- `@playform/compress` for automatic Brotli/gzip compression of all output files.

### 3.7 Alternate URLs

- `.md.ts` endpoints for machine-readable versions of key pages.
- Pricing, tools, compare, and research pages all have `.md` alternates.

### 3.8 Content Collections

- Explicit `slug` field in frontmatter (not filename-derived).
- `content.config.ts` with Zod schemas for validation.
- Use `getCollection()` and `getEntry()` API for type-safe access.

```yaml
---
title: "My Blog Post"
slug: "my-blog-post" # explicit, not derived from filename
publishDate: 2026-03-15
tags: ["email", "security"]
---
```

### 3.9 MDX Components

Reusable components for content pages, exported from `components/mdx/index.ts`:

- `CTA` — call-to-action blocks
- `Callout` — info/warning/tip boxes
- `StepList` — numbered step sequences
- `CodeBlock` — syntax-highlighted code
- `Figure` — images with captions
- `DataTable` — structured data display
- `DashboardShot` — product screenshot with frame

### 3.10 Blog Features

- Table of Contents: auto-generated from headings
- Share buttons: social platform sharing
- Read time: word count-based calculation
- Tags: with dedicated tag index pages (`/blog/tags/[tag]`)
- Search: Pagefind integration on blog index

### 3.11 OG Images

- Dynamic generation via `satori` in `src/og/` directory.
- Template per content type: blog, learn, tools, compare, research.
- Reuse brand colors from design tokens. Font loading via `fonts.ts`.

### 3.12 Design Token Reuse

- Same `shared/design-tokens.css` used by both Astro and SPA.
- Import in Astro's `BaseLayout.astro`. Ensures visual consistency.

### 3.13 Layout System

- `BaseLayout.astro`: HTML skeleton, SEO meta, global styles.
- Specialized layouts per content type:
  - `BlogLayout.astro`, `ToolLayout.astro`, `LearnLayout.astro`, `ResearchLayout.astro`.

### 3.14 Vue Islands

- Interactive components rendered as Vue islands.
- Use `client:visible` (lazy) or `client:load` (eager) directives.
- All tools are Vue components with the `DomainToolForm.vue` + `useDomainTool.ts` pattern.

### 3.15 Trailing Slashes

- Enforced in Astro config: `trailingSlash: "always"`.
- All internal links must end with `/`.

### 3.16 Shared Data

- `shared/site.ts` + `shared/pricing.ts` imported by both Astro and SPA.
- Single source of truth for brand facts, protocol counts, pricing.

### 3.17 Compare Pages

- Data-driven competitor comparison pages.
- TypeScript data files in `src/data/compare/` with shared types.
- DRY pricing token replacement via utility function.

### 3.18 Research Pages

- Reusable research components: `StatCard`, `ProtocolBar`, `DataHighlight`, `KeyFindings`, `MethodologySection`.
- Data-driven layouts for study results.

### 3.19 Print Styles

- Dedicated `print.css` for print-friendly output.

### 3.20 Snippets

- Code/data snippets in `src/snippets/` directory.
- Imported into MDX posts to keep content files clean and focused.

---

## Part 4: Project Structure & Tooling

### 4.1 Linting

- Go: `golangci-lint` v2 with strict config.
  - `depguard` for import control (block global zerolog).
  - `errcheck` with `check-blank: true`.
  - `gocritic`, `gosec`, `bodyclose`, `sqlclosecheck`, `nilerr`.
  - Config format: `version: "2"`, `linters.default: standard`, explicit `enable/disable` lists.

### 4.2 Pre-commit

- `gitleaks` for secret detection.
- `commitlint` for conventional commits (`feat:`, `fix:`, `chore:`, etc.).
- `gofmt`, `go vet`, `goimports` for Go formatting.
- `prettier` for frontend formatting.
- `oxlint` for fast JavaScript/TypeScript linting.

### 4.3 CI Pipeline

- `golangci-lint` GitHub Action (fails only on new issues via `--new-from-rev`).
- Security job: `govulncheck` + `gosec` + `trivy fs`.
- Frontend checks: `eslint` + `astro check`.
- Scheduled: `trivy image` + `gitleaks --history` + `semgrep`.

### 4.4 Justfile

- Recipe-driven automation (not Makefile).
- Standard recipes: `create-migrate-revision`, `sec-vuln`, `sec-secrets`, `sec-deps`, `sec-gosec`, `sec-all`.

### 4.5 Nix

- `devShell` with all project tools.
- Multi-platform support. Reproducible builds.

### 4.6 Docker

- Multi-stage builds: frontend → mod → backend → `scratch`.
- `CGO_ENABLED=0` for static binary.
- `/data` volume for persistent storage.

### 4.7 Single Binary Deployment

- `//go:embed dist` embeds SPA into Go binary.
- `//go:generate sqlc generate` for DB queries.
- `//go:generate bunx mjml ...` for email templates.
- Final image is `scratch` with just the binary.

### 4.8 Boy Scout Rule

- Leave code better than you found it.
- Suggest DRY improvements when you see duplication.
- Don't adhere to broken patterns — fix them.
- But don't over-refactor: only improve code you're actively touching.

---

## Part 5: meysam81/x Library Ecosystem

Preferred library for cross-cutting Go concerns. All packages follow the functional options pattern.

### chimux — Chi Router Factory

```go
r := chimux.NewChi(
    chimux.WithLoggingMiddleware(),
    chimux.WithLogger(log),
    chimux.WithMetrics(),
    chimux.WithHealthz(),
    chimux.WithHealthEndpoint("/health"),
    chimux.WithLogAllHeaders(),
)
```

Opinionated defaults: CleanPath, RealIP, Recoverer enabled.
Structured logging: status-based log level (5xx→Error, 4xx→Warn), sensitive header masking (Authorization, Cookie, X-API-Key).

### logging — Zerolog Wrapper

```go
type Logger = zerolog.Logger

l := logging.NewLogger(
    logging.WithLogLevel("info"),
    logging.WithColorsEnabled(true),
    logging.WithTimeFormat(time.RFC3339),
)
```

Type alias, not a wrapper struct. ConsoleWriter to stderr, caller info included.

### config — Koanf Config Loader

```go
cfg, err := config.NewConfig(
    config.WithYamlConfig("config.yml"),
    config.WithEnvPrefix("APP_"),
    config.WithDefaults(defaults),
    config.WithUnmarshalTo(&appConfig),
)
```

Layered loading: defaults → JSON → YAML → environment variables. Each layer overrides the previous.

### cryptox — Symmetric Encryption

```go
// Cipher interface.
type Cipher interface {
    Encrypt(plaintext []byte) ([]byte, error)
    Decrypt(ciphertext []byte) ([]byte, error)
}

// AES-256-GCM (default).
cipher, _ := cryptox.NewAESGCM(key)

// ChaCha20-Poly1305.
cipher, _ := cryptox.NewChaCha20(key)

// Password-based (Argon2id + AES-256-GCM).
cipher, _ := cryptox.NewArgon2id(password)
```

### sqlite — SQLite Connection Factory

```go
db, err := sqlite.NewDB(ctx, "app.db",
    sqlite.WithJournalMode("wal"),
    sqlite.WithConnMaxOpen(1),
)
```

Build-tag driver split: `mattn/go-sqlite3` (CGO) vs `modernc.org/sqlite` (pure-Go). Automatic file creation if not exists.

### smtpclient — SMTP Email Client

```go
client, _ := smtpclient.New(host, port, user, pass,
    smtpclient.WithTimeout(30*time.Second),
    smtpclient.WithRetry(3, time.Second),
)
err := client.SendEmail(ctx, smtpclient.Email{
    From:     "noreply@example.com",
    To:       []string{"user@example.com"},
    Subject:  "Hello",
    HTMLBody: "<h1>Welcome</h1>",
})
```

### tracing — OpenTelemetry

```go
tracer, _ := tracing.NewTracer(ctx, &tracing.TracingConfig{
    ServiceName:    "my-service",
    ServiceVersion: "1.0.0",
    Enabled:        true,
}, log)

r.Use(tracer.HTTPMiddleware) // chi middleware
```

Auto-shutdown when context is cancelled. No-op tracer when disabled.

### ratelimit — Redis Rate Limiting

Five algorithms, all via atomic Lua scripts:

- `TokenBucket` — allows bursts up to capacity
- `LeakyBucket` — enforces smooth constant rate
- `SlidingWindow` — rolling window with individual timestamps
- `FixedWindow` — discrete non-overlapping windows
- `DistributedSlidingWindow` — multi-node coordination

### downloader — HTTP File Download

Parallel chunked downloads, resumable transfers, SHA256 verification, exponential backoff retry. Automatic strategy selection based on server capabilities.

### httputils — stdlib HTTP Helpers

Logging middleware for `net/http` (use when not using chi/chimux).

---

## Universal Principles

These apply across all languages and frameworks:

1. **DRY**: Grep before creating utilities. Extract shared code. Never duplicate logic.
2. **12-Factor**: Environment-based config. Stateless processes. Explicit dependencies.
3. **Simplicity First**: Every change as simple as possible. Minimal code impact.
4. **No Laziness**: Find root causes. No temporary fixes. Staff-engineer standards.
5. **Security**: Never expose internal errors. Validate at boundaries. Use `crypto/rand`.
6. **Backend-Driven Logic**: ALL business logic in the backend API. Frontend renders decisions, never makes them. Verdicts, scores, permissions — computed server-side.
7. **Boy Scout**: Leave code better than found. Fix patterns, don't perpetuate broken ones.
8. **Confirm Dependencies**: Before adding ANY dependency, ask: can we build it in-house? Must be production-ready, minimal, actively maintained.
9. **No Estimates**: Don't predict how long tasks will take. Focus on what needs doing.
10. **Verify Before Done**: Never mark a task complete without proving it works. Run tests, check logs, demonstrate correctness.
