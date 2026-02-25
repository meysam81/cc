---
name: seo-research
description: Generate a deep research prompt for Claude.ai to gather facts, stats, expert quotes, and competitor analysis for a blog post topic
model: opus
---

# SEO Deep Research Prompt Generator

Generate a focused, completable prompt for Claude.ai's deep research mode. The prompt must be **lean enough to finish within 30-45 minutes** — not exhaust the 2-hour timeout.

## Inputs

1. **Topic or keyword** — provided by user in their message
2. **Brand voice** — read from `docs/seo/brand-voice.md` (if exists)
3. **Topical clusters** — read from `docs/seo/topical-clusters.md` (if exists)
4. **Output path** (optional) — user may specify a file path; default is `docs/seo/<slug>/prompt.md`

## Article Directory Convention

All per-article files live in `docs/seo/<slug>/` where `<slug>` is derived from the topic:

- Lowercase, hyphenated, 3-5 words (e.g., "DMARC Failed: The Complete Troubleshooting Guide" → `dmarc-failed-troubleshooting-guide`)
- If the user specifies a path in their message, use that instead
- Create the directory if it doesn't exist

Files in this directory across the pipeline:

| File                | Created by            |
| ------------------- | --------------------- |
| `prompt.md`         | `/seo-research`       |
| `research-brief.md` | User (from Claude.ai) |
| `keyword-data.md`   | `/seo-keywords`       |
| `outline.md`        | `/seo-outline`        |
| `draft.md`          | `/seo-write`          |

## Process

### Step 1 — Gather Context

Read `docs/seo/brand-voice.md` if it exists. Extract:

- Industry/niche
- Target audience (ICPs)
- Product context
- Key terminology
- Competitors

If the file doesn't exist, ask the user for: industry, target audience, and 2-3 competitor names.

Read `docs/seo/topical-clusters.md` if it exists. Look for the current topic in the cluster map to understand:

- Which pillar this post belongs to
- What other posts in the cluster it should link to
- The strategic role of this post (TOFU/MOFU/BOFU)
- Phase priority and target audience segment

### Step 2 — Classify What Needs External Research vs. What We Already Know

**CRITICAL**: Not everything needs Deep Research. Split the topic into:

**Skip in the prompt** (we can write these ourselves or look up directly):

- RFC specifications — these are static documents, cite them during writing
- Technical definitions — we know the protocols
- Tool lists and CLI commands — standard knowledge
- DNS record syntax — reference material

**Include in the prompt** (needs real-world, current data from the web):

- Recent statistics and adoption data (last 2 years)
- Real case studies and incident reports
- Competitor article analysis and content gaps
- Current regulatory enforcement status and deadlines
- Expert quotes from published sources
- Community questions and pain points (Reddit, forums)

### Step 3 — Generate a Lean Research Prompt

**Hard constraints for the generated prompt:**

- **Maximum 4-5 research sections** (not 7, not 10, not 12)
- **Explicit quantity caps per section** (e.g., "find 3-5 statistics", not open-ended)
- **Total prompt length: 80-120 lines** (not 300+)
- **No redundant output format spec** — keep it to a 5-line example, not a full template
- **Add a stop condition**: "Once you have 3-5 items per section, move to the next section. Do not exhaustively crawl every possible source."

**Anti-hallucination instructions** (place at the top, keep to 4 lines):

```
IMPORTANT: Cite only verifiable sources with URLs. For statistics, include the exact number, source name, year, and URL. If you can't verify a claim, state "unverified" and move on. Prefer primary sources (research reports, official docs) over blog posts. Flag data older than 2 years.
```

**Research sections — pick 4-5 from this menu based on the topic:**

1. **Statistics & data** (3-5 key stats with sources)
2. **Expert quotes** (2-3 verifiable quotes from named experts)
3. **Case studies** (2-3 real-world examples with outcomes)
4. **Regulatory context** (only if compliance is relevant to the topic)
5. **Competitor content gaps** (analyze top 3 articles, not 5-10)
6. **Community questions** (8-10 from Reddit/forums/PAA, not 15-20)

**Sections to OMIT from the prompt** (handle during seo-write instead):

- RFC/protocol specifications — cite directly during writing
- Diagnostic tools & methods — standard knowledge
- Fix workflows — write from expertise during drafting
- Detailed output format templates — a brief example suffices

### Step 4 — Determine Output Path

1. If the user specified a file path in their message, use that
2. Otherwise, derive a slug from the topic (lowercase, hyphenated, 3-5 words)
3. Output path: `docs/seo/<slug>/prompt.md`
4. Create the directory if it doesn't exist

### Step 5 — Write the Prompt to File

Write the generated research prompt to the output file using the Write tool.

Then tell the user:

```
## How to Use This Prompt

Prompt written to `docs/seo/<slug>/prompt.md`.

1. Open Claude.ai (https://claude.ai)
2. Start a new conversation
3. Click the "Deep Research" button (or use the research mode toggle)
4. Open the prompt file and paste its contents
5. Wait for the research to complete (typically 15-30 minutes)
6. Save the output to `docs/seo/<slug>/research-brief.md` in your project
```

## Output Format

The generated prompt should request research organized as a flat markdown file — **no nested templates or complex structure**. A simple example in the prompt is sufficient:

```markdown
# Research Brief: {{Topic}}

## Key Statistics

- [stat] — Source: [name] ([year]) [URL]

## Expert Quotes

- "[quote]" — [name], [title] — Source: [URL]

## Case Studies

### [Name]

- What happened, root cause, resolution, outcome — Source: [URL]

## Content Gaps (vs top 3 ranking articles)

- What they cover: ...
- What they miss: ...

## Community Questions

1. [question] — [source URL]
```

## Quality Checks

Before outputting, verify the prompt:

- [ ] Total prompt is 80-120 lines (not 200+)
- [ ] Maximum 4-5 research sections
- [ ] Each section has explicit quantity caps (3-5 items, not open-ended)
- [ ] Stop condition is present ("move to next section after N items")
- [ ] No RFC/protocol spec requests (handle during writing)
- [ ] No detailed output template (brief example only)
- [ ] Anti-hallucination instructions are concise (4 lines max)
- [ ] Brand context is included but brief (3-4 lines)
