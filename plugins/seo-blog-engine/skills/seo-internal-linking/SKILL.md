---
name: seo-internal-linking
description: Audit all site pages and generate readability-first internal linking recommendations. Uses subagents for content exploration, then orchestrates keyword mapping and link placement.
---

# Internal Linking Audit & Recommendations

Perform a comprehensive internal linking analysis across all content pages on the site. Discover every page, map keywords and topics, identify linking opportunities where they genuinely serve the reader, and produce an actionable report.

**North star**: Every recommended link must improve readability. If following the link doesn't help the reader at that moment, don't recommend it.

## References

Read before starting (relative to this skill's base directory):

- `../../references/internal-linking-guide.md`
- `../../references/on-page-seo-checklist.md`
- `../../references/eeat-signals.md`

## Inputs

1. **Content directory** — user provides path, OR default to `./landing-page/` if present, OR current working directory
2. **Topical clusters** — `docs/seo/topical-clusters.md` (optional, improves link mapping significantly)
3. **Brand voice** — `docs/seo/brand-voice.md` (optional, used for niche/audience context)
4. **Existing keyword data** — `docs/seo/*/keyword-data.md` (optional, used when available to enrich page understanding)

## Content Discovery

Detect the site framework and locate content files:

| Framework    | Content Location                 | File Types         |
| ------------ | -------------------------------- | ------------------ |
| Astro        | `src/content/`, `src/pages/`     | `.md`, `.mdx`      |
| Hugo         | `content/`                       | `.md`              |
| Next.js      | `app/`, `pages/`                 | `.md`, `.mdx`      |
| Gatsby       | `src/pages/`, `content/`         | `.md`, `.mdx`      |
| Jekyll       | `_posts/`, `_pages/`             | `.md`, `.markdown` |
| VitePress    | `docs/`, root `.md` files        | `.md`              |
| Nuxt Content | `content/`                       | `.md`, `.yaml`     |
| Eleventy     | `src/`, root content dirs        | `.md`, `.njk`      |
| Generic      | Scan for `.md`/`.mdx` in project | `.md`, `.mdx`      |

**Detection order**: Check `package.json` or config files (`astro.config.*`, `hugo.toml`, `next.config.*`, etc.) for framework identification. Fall back to directory structure heuristics.

Also check `docs/seo/*/draft.md` — these are pipeline-produced articles that may or may not be in the content directory yet.

## Process

### Phase 1 — Content Exploration (MANDATORY — NON-NEGOTIABLE)

This phase MUST be thorough. Skimming is not acceptable. The quality of every subsequent step depends entirely on how well we understand the content.

**Dispatch exploration subagent(s) via the Task tool.**

For sites with **≤ 20 content pages**: dispatch a single Explore subagent.
For sites with **> 20 content pages**: split by directory, cluster, or content type — one subagent per group.

Each exploration subagent must extract per page:

| Field                   | Source                                        |
| ----------------------- | --------------------------------------------- |
| File path               | File system path                              |
| URL slug                | Derived from file path and framework routing  |
| Title                   | H1 or frontmatter `title`                     |
| Primary keyword         | From frontmatter, first H1, or inferred topic |
| Secondary keywords      | From H2 headings                              |
| Content summary         | 2-3 sentence summary of what the page covers  |
| Search intent           | Informational / Commercial / Transactional    |
| Funnel position         | TOFU / MOFU / BOFU                            |
| Word count              | Approximate                                   |
| Existing internal links | List of (anchor text → target URL) pairs      |
| Existing external links | Count                                         |
| Heading structure       | H1, H2, H3 hierarchy                          |

**Subagent prompt template** (adapt as needed):

> Read every content file in `{{directory}}`. For each file, extract: file path, URL slug (infer from framework routing conventions), title (H1 or frontmatter), primary keyword, secondary keywords (from H2s), a 2-3 sentence content summary, search intent (informational/commercial/transactional), funnel position (TOFU/MOFU/BOFU), approximate word count, all existing internal links as (anchor text → target URL) pairs, external link count, and full heading structure (H1/H2/H3). Return results as a structured list, one entry per page. Be thorough — read each file completely, do not skim.

### Phase 2 — Keyword-to-Page Mapping

With all exploration data collected, build the mapping yourself (do NOT delegate this):

1. **Page inventory table** — all pages with: slug, title, primary keyword, word count, incoming internal link count, outgoing internal link count

2. **Keyword overlap matrix** — identify pages sharing primary/secondary keywords. Flag potential cannibalization (two pages targeting the same primary keyword)

3. **Topical cluster alignment** — if `topical-clusters.md` exists:
   - Map each discovered page to its pillar
   - Identify pages not assigned to any cluster (potentially orphaned topics)
   - Verify pillar pages exist and have supporting posts linking to them

4. **Semantic relationship scoring** — for every potential source→target page pair, evaluate:
   - Keyword overlap (shared primary/secondary terms)
   - Topic continuity (does the target deepen the source's narrative?)
   - Search intent alignment (same or adjacent intent)
   - Reader journey logic (would a reader naturally want this next?)
   - Cluster proximity (same pillar, adjacent pillar, or unrelated)

### Phase 3 — Link Recommendations

For each recommended link, provide:

- **Source page**: file path + slug
- **Target page**: file path + slug
- **Suggested anchor text**: natural, descriptive phrase
- **Placement context**: the paragraph or sentence where the link should appear (quote existing text and show where the link goes)
- **Readability justification**: one sentence explaining WHY this link helps the reader at this specific point
- **Relevance score**: High / Medium (never recommend Low-relevance links)

**Filtering rules:**

- Only recommend links scoring High on at least one factor from the semantic relevance table in the reference guide
- Never recommend more links than the density guidelines suggest for the page's word count
- Never link the same target more than once from the same page
- Skip links where the anchor text would feel forced or unnatural
- If a page already links to the target, don't recommend a duplicate

### Phase 4 — Orphan Detection

Identify pages with:

- **Zero incoming internal links** — critical orphans that crawlers may never discover
- **Only 1 incoming link** — fragile pages dependent on a single link path

For each orphan, recommend the most contextually relevant page to link FROM.

### Phase 5 — Generate Report

Write to `docs/seo/internal-linking-report.md`:

```markdown
# Internal Linking Report

**Date**: {{YYYY-MM-DD}}
**Site**: {{project directory or site name}}
**Pages analyzed**: {{N}}
**Framework detected**: {{framework name}}

## Page Inventory

| #   | Slug     | Title     | Primary Keyword | Words | Incoming Links | Outgoing Links |
| --- | -------- | --------- | --------------- | ----- | -------------- | -------------- |
| 1   | {{slug}} | {{title}} | {{keyword}}     | {{N}} | {{N}}          | {{N}}          |
| ... |          |           |                 |       |                |                |

## Orphan Pages ⚠️

Pages with zero incoming internal links:

| Page      | Slug     | Recommended Link From | Suggested Anchor |
| --------- | -------- | --------------------- | ---------------- |
| {{title}} | {{slug}} | {{source page}}       | {{anchor text}}  |
| ...       |          |                       |                  |

{{If no orphans: "✅ No orphan pages detected — all pages have at least one incoming internal link."}}

## Recommended Links

### {{Source Page Title}} ({{slug}})

Current: {{N}} outgoing links | Recommended additions: {{N}}

**Link 1**:

- **Target**: [{{target title}}]({{target slug}})
- **Anchor text**: "{{suggested anchor}}"
- **Placement**: In the paragraph: "...{{surrounding text with [anchor text](target) inserted}}..."
- **Why**: {{readability justification}}

**Link 2**:
...

---

### {{Next Source Page}}

...

## Summary

| Metric                                       | Value          |
| -------------------------------------------- | -------------- |
| Total pages analyzed                         | {{N}}          |
| Orphan pages found                           | {{N}}          |
| New links recommended                        | {{N}}          |
| Pages receiving new links                    | {{N}}          |
| Average link density (current)               | {{N}} per page |
| Average link density (after recommendations) | {{N}} per page |

{{If topical-clusters.md was NOT found:}}

## Suggested Topical Cluster Map

Based on the content analysis, here's a suggested cluster structure:

### Pillar: {{Topic}}

- {{Pillar page slug}}
  - {{Supporting page 1 slug}}
  - {{Supporting page 2 slug}}

### Pillar: {{Topic}}

...

{{End conditional}}
```

### Phase 6 — Apply Links (after user confirmation)

**Wait for the user to review the report.** Do NOT apply changes automatically.

When the user confirms:

1. For each recommended link, edit the source file:
   - Find the exact paragraph from the placement context
   - Insert the markdown link at the specified anchor text location
   - Preserve all existing formatting, frontmatter, and content

2. If the URL slug pattern is unclear (e.g., the framework uses a custom routing config you can't parse), **stop and ask the user** for the correct URL format before inserting links.

3. After applying, summarize: "Applied {{N}} internal links across {{N}} pages."

## Quality Checks

Before saving the report, verify:

- [ ] Every recommended link has a readability justification (not just "related topic")
- [ ] No page exceeds the density guidelines for its word count
- [ ] No target is linked more than once from the same source page
- [ ] All orphan pages have a recommended incoming link
- [ ] Anchor text is natural and descriptive (no "click here", no keyword stuffing)
- [ ] If topical clusters exist, pillar ↔ supporting links are prioritized
- [ ] Existing links are accounted for (don't recommend links that already exist)

## User Action

After generating the report, tell the user:

> Review the recommendations in `docs/seo/internal-linking-report.md`. Each link includes the exact placement and a readability justification. When you're ready, confirm and I'll apply the approved links to your content files. You can also cherry-pick specific links to apply.
