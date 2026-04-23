---
name: seo-batch
description: Orchestrate a phase-gated, parallel-fan-out batch of N SEO blog posts across specified publish dates. Use when the user asks to plan and ship multiple SEO posts for a set of dates (e.g., "write the next 4 blog posts for Apr 20, 23, 24, 27"). Reads a requirements file or inline dates, coordinates the seo-keywords, seo-research, seo-outline, seo-write, seo-review, and seo-optimize skills with human review gates between phases.
---

# SEO Content Batch Orchestrator

Ship N SEO-optimized blog posts across specified publish dates through a semi-autonomous, phase-gated pipeline. Composes the single-step skills in this plugin (`seo-keywords`, `seo-research`, `seo-outline`, `seo-write`, `seo-review`, `seo-optimize`) into a batch workflow with parallel fan-out per phase and human review gates between phases.

The human-in-loop gates are load-bearing for E-E-A-T quality. Do not automate them away.

## When to Use

Trigger this skill when the user asks for any of:

- "Write the next N blog posts for dates X, Y, Z..."
- "Ship a batch of SEO posts targeting [topic cluster] on [dates]"
- "Plan, research, and publish [N] articles by [deadline]"
- Provides a requirements file (e.g., `seo-upcoming-work.md`) describing a multi-post batch

For a SINGLE post, use the individual skills directly (`seo-keywords` → `seo-research` → `seo-outline` → `seo-write` → `seo-review`). This skill only wins on batches of 2 or more posts where cross-post consistency and parallelism matter.

## Inputs

1. **Requirements file** — path to a markdown file describing the batch. Default lookup order: `seo-upcoming-work.md`, `docs/seo/upcoming-batch.md`, `.claude/tasks/seo-batch.md`. If none exists, ask the user for inline dates + any E-E-A-T constraints.
2. **Publish dates** — N dates (one per post). Extract from the requirements file or ask inline.
3. **Queue tracker** — `docs/seo/next.md` (if exists). Use to pick the top N pending posts if the user does not specify topics.
4. **Brand voice** — `docs/seo/brand-voice.md` (MUST exist). Red lines, voice rules, CTA templates.
5. **SEO conventions** — `docs/seo/seo-conventions.md` (MUST exist). Hard rules: hero/OG image pattern, FAQ format, RFC citation pattern, DataForSEO persistence.
6. **Content strategy** — `docs/seo/content-strategy-next-10.md` OR equivalent. Pillars, unique angles per queued post.
7. **First-party research inventory** — `research-data/` and `landing-page/src/pages/research/` (if present). E-E-A-T anchor.
8. **Chat transcripts for niche context** — `chatgpt.md`, `gemini.md`, or similar if referenced in the requirements file.
9. **Internal link inventory** — verify-by-grep against `landing-page/src/content/blog/` for actual paths that exist.

## References

Read these shared references alongside the per-skill references during execution:

- `../../references/on-page-seo-checklist.md`
- `../../references/eeat-signals.md`
- `../../references/brand-voice-defaults.md` (if exists)

## Hard Rules (enforced at every phase gate)

These inherit from `CLAUDE.md` and `docs/seo/brand-voice.md`. Non-negotiable.

1. **First-party stats always cited.** Every percent or count must have a URL, research-data path, or brief ID next to it.
2. **No estimates or approximations** in user-facing body. If data does not exist, compute from raw data or omit the claim.
3. **No fear-based marketing.** Inform and guide; never alarm.
4. **Never call the product "open-source", "OSS", "auditable", or "self-hosted"** (this applies to DMARCguard specifically; external-project references are fine).
5. **Protocol names always capitalized** (DMARC, SPF, DKIM, ARC, BIMI, MTA-STS, DANE, TLS-RPT, etc.).
6. **RFC first-mention convention** — cite `RFC NNNN` inline on first body mention of each protocol, then use the bare protocol name. Applies to RFC 7489 (DMARC), 7208 (SPF), 6376 (DKIM), 8617 (ARC), 8461 (MTA-STS), 8460 (TLS-RPT), etc.
7. **DataForSEO MCP outputs MUST be persisted** to `docs/seo/<slug>/keyword-data.md` including raw payloads. Paid API — do not discard.
8. **Author field is forbidden in frontmatter.** `BlogLayout` defaults to the team name. Do not set it per post.
9. **Hero/OG image is auto-generated from slug** — frontmatter MUST NOT declare `image:` or `imageAlt:`. The route `/og/blog/<slug>.png` is built at deploy time.
10. **seo-review score must be ≥80/100** before publishing. If a draft scores lower, iterate.

## Execution Model

Orchestrator (main context) dispatches N parallel subagents per phase, reconciles outputs at a review gate, then advances.

```
Phase 0 — Bootstrap                                  (orchestrator only)
Phase 1 — Keyword refresh         (N ∥ subagents, seo-keywords)
   └── GATE 1: user reviews N × keyword-data.md
Phase 2 — Research prompts        (N ∥ subagents, seo-research)
   └── GATE 2: user runs prompts on claude.ai, writes briefs back
Phase 3 — Outlines                (N ∥ subagents, seo-outline) + link reconciliation
   └── GATE 3: user reviews N × outline.md
Phase 4 — Drafts                  (N ∥ subagents, seo-write)
   └── GATE 4: user reviews N × draft.md
Phase 5 — Review + publish        (N ∥ subagents, seo-review → MDX + snippets + SVG figures)
   └── GATE 5: user confirms N × MDX + build checks
Phase 6 — Post-publish verification                  (orchestrator only)
```

### Gate protocol (every gate follows the same shape)

At each gate the orchestrator MUST:

1. Read every artifact produced in the phase.
2. Spot-check for red-line violations (grep for banned voice words, unsourced stats, dead internal links).
3. Post a ≤200-word summary to the user: what changed, any anomalies, any judgement calls that warrant review.
4. STOP. Wait for the user to reply with an explicit "proceed", "approve", or a redirection. Do NOT dispatch the next phase until the user has responded.

If the user asks for changes, apply them in-place (or re-dispatch a targeted subagent), re-run the gate checks, present the revised summary, and wait again.

### Subagent dispatch rules

- Use `general-purpose` subagent type (has Read/Edit/Write/Glob/Grep + Bash). Do NOT use a `Bash`-only subagent — file creation tasks fail under the permission model.
- Dispatch N subagents in a SINGLE message with N `Agent` tool calls. This runs them concurrently.
- Each subagent prompt is self-contained: slug, target date, primary+secondary KWs, unique angle, verified internal-link inventory, voice rules summary, file paths to read and write. See "Reusable Subagent Prompt Template" below.
- Run `bunx prettier -w` against the phase's outputs before the gate to avoid formatter noise.
- Prefer `run_in_background: true` on each Agent dispatch; you will be notified as each completes.

## Phase 0 — Bootstrap

Before dispatching anything, the orchestrator MUST:

1. Read the requirements file. Extract: N (post count), publish dates, any topic pre-selections, E-E-A-T must-haves (first-party stat anchors), special-scope notes (e.g., "DKIM2 draft callout is scope-bounded to one post only").
2. Read the mandatory files listed under "Inputs".
3. Verify the file structure of the `seo-blog-engine` artifact convention exists (see "Per-Post Artifact Layout" below). Create any missing directories.
4. Derive the verified internal-link inventory: grep existing blog MDX files for `/(learn|tools|compare|blog|research)/[a-z0-9-]+/?` paths. Record the set; subagents may only link to paths in this set (plus cross-batch forward links to sibling posts in THIS batch).
5. Consult `docs/seo/next.md` or `docs/seo/content-strategy-next-10.md` to map the N publish dates to N planned posts. Confirm the mapping with the user BEFORE dispatching Phase 1.

Gate 0 is a confirmation gate ("Ready to dispatch Phase 1 for slugs [A, B, C, D] on dates [W, X, Y, Z] — proceed?"). Wait for confirmation.

## Phase 1 — Keyword refresh

Dispatch N parallel subagents. Each subagent invokes the `seo-keywords` skill for its assigned slug.

Subagent body instructs: overwrite `docs/seo/<slug>/keyword-data.md` with refreshed DataForSEO data; preserve the baseline structure; annotate any KW that is stable vs the baseline; persist raw DataForSEO JSON at the bottom.

Orchestrator gate: summarize volume/KD/SERP changes vs the previous refresh; flag any KW whose topic assumption has shifted (e.g., a CPC spike/collapse, a SERP top-10 churn). Recommend reframes if warranted. Wait for user "proceed".

## Phase 2 — Research prompts (human-in-loop)

Dispatch N parallel subagents. Each subagent invokes the `seo-research` skill and generates 3–5 research prompts per post, written as paste-ready claude.ai Deep Research inputs.

Scope-bounded sub-topics (e.g., an emerging IETF draft that enters as a single callout) MUST be captured as ONE dedicated prompt with an explicit "this feeds a single callout — not primary thesis" framing in the prompt body.

Gate 2 is the long gate: orchestrator presents the full list of prompts; user runs each on claude.ai Deep Research and saves responses to `docs/seo/<slug>/research-brief-N-<topic-slug>.md` matching the prompt numbering. Briefs may land asynchronously. Wait for explicit "proceed" (plus verify ≥1 brief per post exists) before Phase 3.

## Phase 3 — Outlines + cross-post reconciliation

Dispatch N parallel subagents. Each subagent invokes `seo-outline` for its slug.

After all N return, the orchestrator MUST reconcile:

- Every internal link target in every outline must exist (grep the blog collection) OR be a cross-batch forward link to a sibling post in THIS batch (its slug is known).
- Cross-post reciprocal links: pair posts within the batch that share a cluster (e.g., compliance pair, troubleshooting pair) and ensure they link to each other.
- Each outline has 3–5 internal links in body prose; extras are OK only if they anchor tool-CTA components.

Gate 3: summarize title/meta char counts, H2 counts, target word counts, FAQ counts, and any broken/orphan links fixed. Wait for "proceed".

## Phase 4 — Drafts

Dispatch N parallel subagents. Each subagent invokes `seo-write` for its slug.

Orchestrator spot-checks the drafts with a grep scan for banned voice words, unsourced percent stats, forbidden `author:` field, manual JSON-LD blocks, and `image:`/`imageAlt:` frontmatter fields (all four are red lines).

Gate 4: report word counts, stat-citation counts, any red-line violations that required fix-up, any drafts materially over/under the outline's target word count. Wait for "proceed".

## Phase 5 — Review + publish

Dispatch N parallel subagents. Each subagent MUST:

1. Invoke `seo-review` against `docs/seo/<slug>/draft.md`. Score against the 100-point rubric; write `docs/seo/<slug>/review.md`. Iterate the draft until score ≥80. Do NOT publish a draft that scores below 80.
2. Convert `draft.md` → MDX at the assigned path `landing-page/src/content/blog/000000N-<slug>.mdx` (auto-number, increment from the current max). Follow the plugin's precedent patterns (BlogLayout-compatible frontmatter, Figure/Callout/CodeBlock components, snippet imports for multi-line code).
3. Author any SVG figures at `landing-page/public/images/blog/<slug>/<slug>_<desc>_<type>.svg` following the seo-conventions SVG rules (hardcoded hex palette, `role="img"` + `aria-labelledby`, shadow filters, arrow markers).
4. Externalize any code block >1 line to `landing-page/src/snippets/<slug>/<topic>.<ext>`.
5. DO NOT author a hero PNG. DO NOT add `image:` or `imageAlt:` to frontmatter. The OG image is auto-generated from the slug.
6. Run `bunx astro check` scoped to the new MDX; run `bunx prettier -w` on every edited file.

Gate 5: report MDX paths, seo-review scores, SVG/snippet inventory, astro-check result. Wait for "proceed".

## Phase 6 — Post-publish verification

Orchestrator-only (no fan-out):

1. Run `bun run build` in the `landing-page/` directory.
2. Verify `dist/sitemap-0.xml` and `dist/rss.xml` include the newly-published posts (future-dated posts are correctly excluded and will appear on their publish dates via scheduled builds).
3. Verify `dist/pagefind/` indexed the live posts.
4. Verify `dist/og/blog/<slug>.png` was generated for every live post. Confirm dimensions are 2400×1260 (2x retina).
5. Update `docs/seo/next.md` queue tracker: move completed posts to the "Published" table with their publish dates and any deviation notes.
6. Produce a final report: slugs + publish dates + seo-review scores, cross-post linking summary, suggested next batch.

## Per-Post Artifact Layout

```
docs/seo/<slug>/
  keyword-data.md          # Phase 1
  prompt-N-<topic>.md      # Phase 2 (3–5 files per post)
  research-brief-N-<topic>.md  # Phase 2 (user-written; paired 1:1 with prompts)
  outline.md               # Phase 3
  draft.md                 # Phase 4 pre-MDX prose
  review.md                # Phase 5 rubric score + notes

landing-page/src/content/blog/
  000000N-<slug>.mdx       # Phase 5 (auto-numbered)

landing-page/src/snippets/<slug>/
  <topic>.<ext>            # Phase 5 multi-line code externalization

landing-page/public/images/blog/<slug>/
  <slug>_<desc>_<type>.svg # Phase 5 in-body SVG figures (no hero PNG)
```

## Reusable Subagent Prompt Template

Every subagent dispatch uses this skeleton. Phase-specific bodies slot into `{{PHASE_BODY}}`. Per-post values fill `{{SLUG}}`, `{{PUBLISH_DATE}}`, `{{PILLAR}}`, `{{PRIMARY_KW}}`, `{{SECONDARY_KWS}}`, `{{UNIQUE_ANGLE}}`.

```
You are a senior SEO content specialist working in the repo at {{REPO_ROOT}}.

Your assignment: {{PHASE_NAME}} for blog post "{{SLUG}}" (pillar {{PILLAR}}),
publish date {{PUBLISH_DATE}}, primary KW "{{PRIMARY_KW}}", secondary KWs
{{SECONDARY_KWS}}. Unique angle: {{UNIQUE_ANGLE}}.

MUST READ (in this order, before doing anything else):
1. {{REQUIREMENTS_FILE}} (this batch's north-star doc)
2. docs/seo/brand-voice.md (voice, red lines, CTAs, terminology)
3. docs/seo/seo-conventions.md (hard rules: image pattern, FAQ format, RFC citations, DataForSEO persistence)
4. docs/seo/content-strategy-next-10.md or equivalent (your post's unique angle + SERP notes)
5. CLAUDE.md project file (hard rules the project enforces)
6. docs/seo/<slug>/<artifacts from prior phases> — read whichever are already in place

FIRST-PARTY DATA SOURCES (cite concrete numbers, never estimate):
{{FIRST_PARTY_STAT_MAP}}

BRAND-VOICE RED LINES (non-negotiable):
- Never call the product "open-source", "OSS", "auditable", or "self-hosted"
- Never use "simple", "easy", "intuitive", "seamless", "enterprise-grade", "robust", "powerful platform", or other banned words from brand-voice.md
- Never use fear-based marketing
- Never use estimates — cite sources for every stat
- Author field is forbidden in frontmatter (BlogLayout defaults to team name)
- Image/imageAlt frontmatter fields are forbidden — OG is auto-generated from slug
- Protocol names always capitalized; RFC first-mention for each protocol

VERIFIED INTERNAL LINK TARGETS (use ONLY these — do not invent paths):
{{VERIFIED_LINK_INVENTORY}}

CROSS-POST LINKS (this batch — will exist by publish date):
{{BATCH_CROSS_LINKS}}

{{PHASE_BODY}}

OUTPUT CONTRACT: Return a short report (under 200 words) summarizing what you
did, where you wrote it, and any blockers. The orchestrator will re-read your
artifacts at the phase gate.
```

## Success Signals

By the end of the session the user has:

- N published MDX files at `landing-page/src/content/blog/` with correct `publishedAt` dates
- N sets of per-post artifacts under `docs/seo/<slug>/`
- SVG figures and snippet files where the drafts called for them
- A build that surfaces the live posts in sitemap/RSS/pagefind and auto-generates OG images
- An updated queue tracker in `docs/seo/next.md`
- A final report with seo-review scores, first-party stat map, and suggested next batch

## Failure Modes to Avoid

- **Dispatching phases in parallel across posts before the prior phase's gate.** N parallel within a phase is correct. N parallel across phases is not — it breaks cross-post reconciliation.
- **Automating the claude.ai Deep Research step.** The human gate is E-E-A-T insurance. Do not replace with WebSearch-only subagents.
- **Inventing internal link paths.** Every link must grep-resolve to an existing file OR be a batch-sibling forward link.
- **Letting a subagent author a hero PNG.** Frontmatter must not declare `image:`; OG is slug-derived.
- **Skipping `seo-review` to hit a deadline.** Score ≥80 is a hard gate. If a post cannot reach 80, escalate to the user before publishing.
- **Committing.** This skill never runs `git commit`. File writes only.
