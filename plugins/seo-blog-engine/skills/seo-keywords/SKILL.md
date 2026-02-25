---
name: seo-keywords
description: Perform keyword research and SERP analysis using DataForSEO MCP tools. Analyzes search volume, difficulty, intent, trends, and competitor gaps.
model: opus
tools:
  - mcp__dataforseo__dataforseo_labs_google_keyword_suggestions
  - mcp__dataforseo__dataforseo_labs_google_keyword_ideas
  - mcp__dataforseo__dataforseo_labs_search_intent
  - mcp__dataforseo__dataforseo_labs_bulk_keyword_difficulty
  - mcp__dataforseo__dataforseo_labs_google_keyword_overview
  - mcp__dataforseo__kw_data_dfs_trends_explore
  - mcp__dataforseo__serp_organic_live_advanced
  - mcp__dataforseo__dataforseo_labs_google_domain_intersection
  - mcp__dataforseo__on_page_content_parsing
  - mcp__dataforseo__backlinks_summary
  - mcp__dataforseo__dataforseo_labs_google_ranked_keywords
  - mcp__dataforseo__dataforseo_labs_google_related_keywords
  - mcp__dataforseo__dataforseo_labs_google_serp_competitors
  - mcp__dataforseo__dataforseo_labs_google_relevant_pages
  - mcp__dataforseo__ai_opt_kw_data_loc_and_lang
---

# SEO Keyword Research & SERP Analysis

Perform comprehensive keyword research using DataForSEO MCP tools. Outputs a structured keyword data file for use by `seo-outline` and `seo-write` skills.

## Reference

Read `../../references/dataforseo-workflows.md` (relative to this skill's base directory) for exact tool parameters and sequences.

## Inputs

1. **Seed keyword(s)** — provided by user
2. **Brand voice** — read `docs/seo/brand-voice.md` for niche context
3. **Topical clusters** — read `docs/seo/topical-clusters.md` (if exists) to understand pillar context, related posts, and strategic role
4. **Article directory** — user specifies `docs/seo/<slug>/` or the skill infers it from the seed keyword
5. **Target market** — default US (location_code: 2840, language_code: "en"); ask if different

## Article Directory Convention

All per-article files live in `docs/seo/<slug>/`. If the user provides a slug or article directory, use it. Otherwise, derive from the seed keyword (lowercase, hyphenated, 3-5 words). If a `prompt.md` already exists in the directory (from `/seo-research`), that confirms the correct directory.

## Process

### Step 1 — Seed Expansion (run in parallel)

Use `keyword_suggestions` AND `keyword_ideas` with the seed keyword(s).

- Location: 2840 (US default)
- Limit: 50 each
- Merge results, deduplicate by keyword string

### Step 2 — Intent Classification

Use `search_intent` on the merged keyword list (max 100).

**Filter rules:**

- KEEP: `informational`, `commercial_investigation`
- FLAG: `transactional` (may be better as product/landing page, not blog)
- DROP: `navigational` (skip for blog content)

### Step 3 — Difficulty Assessment

Use `bulk_keyword_difficulty` on filtered keywords.

**Categorize and advise:**
| KD Range | Label | Guidance |
|----------|-------|----------|
| 0-30 | Easy | Good target for any domain |
| 31-50 | Medium | Target for domains with some authority |
| 51-70 | Hard | Need strong backlink strategy |
| 71+ | Very hard | Skip unless domain has high DA; suggest alternatives |

If primary keyword is KD > 70, proactively suggest lower-difficulty alternatives from the expanded list.

### Step 4 — Detailed Validation

Use `keyword_overview` on the top 20 candidates.

Extract: search_volume, cpc, competition_level, clickstream data.

**Reject if:**

- Search volume < 50/mo for B2B topics
- Search volume < 500/mo for B2C topics
- CPC = 0 AND competition = 0 (likely no commercial value)

### Step 5 — Trend Check

Use `dfs_trends_explore` for the primary keyword and top 3 secondaries.

- Date range: last 12 months
- **Reject** if declining > 30% over 12 months
- **Flag** seasonal patterns (note peak months)
- **Highlight** rising trends (> 20% growth)

### Step 6 — SERP Analysis

Use `serp_organic_live_advanced` for the primary keyword.

- Depth: 10 (top 10 results)
- Device: desktop

For each of the top 5 results, use `on_page_content_parsing` to extract:

- Heading structure (H1, H2, H3)
- Word count
- Internal/external link count

Use `backlinks_summary` for each top 5 URL to get referring domain count.

### Step 7 — Competitor Gap Analysis (if competitors known)

If `brand-voice.md` lists competitors, use `domain_intersection` to find keywords they rank for that you don't.

Also use `related_keywords` and `serp_competitors` to identify additional opportunities.

Use `relevant_pages` to check if your domain already has a page that could rank (update vs. create new).

## Output

Write results to `docs/seo/<slug>/keyword-data.md`:

```markdown
# Keyword Research: {{Primary Keyword}}

**Date**: {{YYYY-MM-DD}}
**Market**: {{US/UK/etc}} | **Seed**: {{original seed keyword}}

## Primary Keyword

| Metric             | Value                               |
| ------------------ | ----------------------------------- |
| Keyword            | {{keyword}}                         |
| Search Volume      | {{monthly}}                         |
| Keyword Difficulty | {{KD score}} ({{Easy/Medium/Hard}}) |
| CPC                | ${{value}}                          |
| Intent             | {{informational/commercial}}        |
| Trend              | {{Rising/Stable/Declining}}         |

**Rationale**: {{Why this keyword was selected as primary}}

## Secondary Keywords

Map to planned H2 headings:

| Keyword | Volume  | KD     | Intent     | Target H2   |
| ------- | ------- | ------ | ---------- | ----------- |
| {{kw1}} | {{vol}} | {{kd}} | {{intent}} | {{heading}} |
| ...     |         |        |            |             |

## LSI / Related Terms

For natural distribution in body text:

| Term      | Relevance       |
| --------- | --------------- |
| {{term1}} | {{High/Medium}} |
| ...       |                 |

## SERP Features

| Feature          | Present?   |
| ---------------- | ---------- |
| Featured Snippet | {{Yes/No}} |
| People Also Ask  | {{Yes/No}} |
| Video Results    | {{Yes/No}} |
| Image Pack       | {{Yes/No}} |
| Knowledge Panel  | {{Yes/No}} |

## Top 5 Competitor Analysis

| #   | URL     | Word Count | H2 Count | Referring Domains |
| --- | ------- | ---------- | -------- | ----------------- |
| 1   | {{url}} | {{wc}}     | {{h2s}}  | {{rd}}            |
| ... |         |            |          |                   |

### Common Heading Structure

{{List the H2 headings that appear across multiple top results}}

## Content Gap Opportunities

{{Topics/angles that top results miss or cover poorly}}

## Backlink Benchmark

- Median referring domains (top 5): {{N}}
- Minimum to compete: {{estimated}}

## Recommendations

- **Content type**: {{Comprehensive guide / How-to / List post / Comparison}}
- **Target word count**: {{range based on competitor analysis}}
- **Difficulty assessment**: {{Achievable/Challenging/Long-term play}}
- **Featured snippet opportunity**: {{Yes/No — format to target}}
```

## Decision Framework

Present these alerts prominently:

- **KD > 70**: "This keyword is very competitive. Consider '{{alternative}}' (KD: {{N}}, Volume: {{N}}) as a more achievable target."
- **Volume < 50/mo (B2B)**: "Low search volume. Viable as a supporting topic in a content cluster, but unlikely to drive significant traffic alone."
- **Declining trend**: "This keyword's search interest has declined {{N}}% over the past 12 months. Consider whether the topic is still relevant."
- **No SERP features**: "No featured snippet or PAA present — harder to stand out, but also less crowded."
- **Existing page found**: "Your domain already has {{URL}} ranking at position {{N}} for related terms. Consider updating that page instead of creating new content."
