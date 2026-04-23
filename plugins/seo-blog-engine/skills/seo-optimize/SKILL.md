---
name: seo-optimize
description: Post-publish validation using DataForSEO tools. Checks Lighthouse scores, on-page SEO verification, ranking position, and AI search visibility.
tools:
  - mcp__dataforseo__on_page_content_parsing
  - mcp__dataforseo__on_page_lighthouse
  - mcp__dataforseo__dataforseo_labs_google_ranked_keywords
  - mcp__dataforseo__content_analysis_search
  - mcp__dataforseo__ai_opt_llm_ment_search
  - mcp__dataforseo__dataforseo_labs_bulk_traffic_estimation
  - mcp__dataforseo__on_page_instant_pages
---

# Post-Publish SEO Validation

Validate a published blog post's on-page SEO, performance, and initial search visibility using DataForSEO tools.

## Reference

Read `../../references/dataforseo-workflows.md` (relative to this skill's base directory) — Workflow 4 (Post-Publish Validation) for exact tool parameters.

## Inputs

1. **Published URL** — provided by user
2. **Primary keyword** — provided by user (or read from `docs/seo/<slug>/keyword-data.md`)
3. **Outline** — `docs/seo/<slug>/outline.md` (optional, for verification against plan)
4. **Article directory** — user specifies `docs/seo/<slug>/` or skill infers from keyword-data.md location

## Article Directory Convention

All per-article files live in `docs/seo/<slug>/`. The user provides the slug, or the skill searches for the directory containing `keyword-data.md` with the matching primary keyword.

## Process

### Step 1 — On-Page Verification

Use `on_page_content_parsing` with the published URL.

**Verify against outline (if available):**

| Element              | Expected              | Actual     | Match? |
| -------------------- | --------------------- | ---------- | ------ |
| Title tag            | {{from outline}}      | {{parsed}} | ✅/❌  |
| Meta description     | {{from outline}}      | {{parsed}} | ✅/❌  |
| H1                   | {{from outline}}      | {{parsed}} | ✅/❌  |
| H2 count             | {{from outline}}      | {{parsed}} | ✅/❌  |
| Word count           | {{from outline ±10%}} | {{parsed}} | ✅/❌  |
| Internal links       | ≥ 3                   | {{parsed}} | ✅/❌  |
| External links       | ≥ 1                   | {{parsed}} | ✅/❌  |
| Images with alt text | ≥ 1                   | {{parsed}} | ✅/❌  |

### Step 2 — Lighthouse Audit

Use `on_page_lighthouse` with mobile device.

**Score targets:**

| Category      | Target | Status                            |
| ------------- | ------ | --------------------------------- |
| SEO           | > 90   | ✅ Good / ⚠️ Needs work / ❌ Poor |
| Performance   | > 50   | ✅/⚠️/❌                          |
| Accessibility | > 80   | ✅/⚠️/❌                          |

Report specific Lighthouse audit failures (missing meta tags, slow LCP, missing alt text, etc.).

### Step 3 — Ranking Check

Use `dataforseo_labs_google_ranked_keywords` filtered for the primary keyword.

**Note**: New content may take days or weeks to be indexed. If not found:

> "Page not yet indexed for '{{primary keyword}}'. This is normal for new content. Re-run `/seo-optimize` in 1-2 weeks to check ranking position."

If found, report:

- Position (rank)
- Estimated traffic
- SERP features triggered (featured snippet, PAA, etc.)

### Step 4 — Content Mentions

Use `content_analysis_search` with the brand name or article title.

Report:

- Number of web mentions found
- Sentiment distribution (positive/neutral/negative)
- Notable mention sources

### Step 5 — AI Search Visibility

Use `ai_opt_llm_ment_search` with the primary keyword.

Check if the content or brand appears in AI-generated search results.

Report:

- Mentioned: Yes/No
- Context of mention (if found)
- Competing mentions

### Step 6 — Traffic Baseline

Use `dataforseo_labs_bulk_traffic_estimation` with the published URL.

Record baseline traffic estimate for future comparison.

## Output

Present the report directly in the conversation:

```markdown
# Post-Publish Health Report

**URL**: {{published URL}}
**Primary keyword**: {{keyword}}
**Date checked**: {{YYYY-MM-DD}}

---

## On-Page Verification

| Element          | Expected        | Actual     | Status |
| ---------------- | --------------- | ---------- | ------ |
| Title tag        | {{expected}}    | {{actual}} | ✅/❌  |
| Meta description | {{expected}}    | {{actual}} | ✅/❌  |
| H1               | {{expected}}    | {{actual}} | ✅/❌  |
| Word count       | {{target ±10%}} | {{actual}} | ✅/❌  |
| Internal links   | ≥ 3             | {{count}}  | ✅/❌  |
| External links   | ≥ 1             | {{count}}  | ✅/❌  |
| Images w/ alt    | ≥ 1             | {{count}}  | ✅/❌  |

**On-page score**: {{N}}/7 checks passing

---

## Lighthouse Scores

| Category      | Score | Target | Status   |
| ------------- | ----- | ------ | -------- |
| SEO           | {{N}} | > 90   | ✅/⚠️/❌ |
| Performance   | {{N}} | > 50   | ✅/⚠️/❌ |
| Accessibility | {{N}} | > 80   | ✅/⚠️/❌ |

**Key issues** (if any):

- {{Lighthouse audit failure 1}}
- {{Lighthouse audit failure 2}}

---

## Search Visibility

| Metric                    | Value               |
| ------------------------- | ------------------- |
| Indexed                   | {{Yes / Not yet}}   |
| Ranking position          | {{#N / Not ranked}} |
| Featured snippet          | {{Yes / No}}        |
| PAA appearance            | {{Yes / No}}        |
| AI search mention         | {{Yes / No}}        |
| Estimated monthly traffic | {{N}}               |

---

## Web Mentions

| Metric          | Value                              |
| --------------- | ---------------------------------- |
| Total mentions  | {{N}}                              |
| Sentiment       | {{Mostly positive / Mixed / None}} |
| Notable sources | {{Source 1, Source 2}}             |

---

## Recommendations

### Immediate Fixes

(Only list if on-page or Lighthouse issues found)

1. {{Specific fix with context}}

### Growth Actions

(Longer-term improvements)

1. {{Action — e.g., build backlinks, create supporting content}}

### Re-check Timeline

- **1 week**: Re-run to check indexing status
- **1 month**: Re-run to check ranking position and traffic
- **3 months**: Full re-evaluation against keyword targets
```

## Notes

- Some checks (ranking, traffic) won't return data for newly published content. This is expected. Note the timeline for re-checking.
- If Lighthouse fails to load the page (e.g., behind auth, geo-restricted), note this and skip to other checks.
- If the outline file isn't available, skip the "Expected" column comparisons and just report what was found on the live page.
