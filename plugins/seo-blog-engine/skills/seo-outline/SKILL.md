---
name: seo-outline
description: Create a detailed content brief and outline from research and keyword data. Generates heading hierarchy, word count targets, keyword assignments, and schema plan.
model: opus
---

# SEO Content Outline Generator

Create a comprehensive, SEO-optimized content outline from research data and keyword analysis. The outline becomes the blueprint for the `seo-write` skill.

## References

Read these before generating the outline (relative to this skill's base directory):

- `../../references/on-page-seo-checklist.md`
- `../../references/eeat-signals.md`
- `../../references/schema-markup-templates.md`

## Inputs

The first three are required. If any is missing, tell the user which file to create first.

1. **Research brief** — `docs/seo/<slug>/research-brief.md` (from `seo-research` → Claude.ai)
2. **Keyword data** — `docs/seo/<slug>/keyword-data.md` (from `seo-keywords`)
3. **Brand voice** — `docs/seo/brand-voice.md`
4. **Topical clusters** — `docs/seo/topical-clusters.md` (if exists) — use to plan internal links to sibling posts in the same pillar and cross-pillar links
5. **Article directory** — user specifies `docs/seo/<slug>/` or skill looks for the most recent article directory with both `research-brief.md` and `keyword-data.md`

## Article Directory Convention

All per-article files live in `docs/seo/<slug>/`. The user provides the slug, or the skill finds it by looking for the directory containing the required input files.

## Process

### Step 1 — Read & Synthesize Inputs

Read all three files. Extract:

- Primary keyword, secondary keywords, LSI terms (from keyword data)
- Top competitor heading structures and word counts (from keyword data)
- Key facts, statistics, expert quotes (from research brief)
- Content gaps identified (from keyword data + research)
- Brand tone, terminology, ICPs (from brand voice)
- SERP features to target (featured snippet, PAA, etc.)

### Step 2 — Determine Content Strategy

Based on competitor analysis and keyword intent:

**Content type** (pick one):

- **Comprehensive guide**: Intent is broad informational, competitors average 2000+ words
- **How-to tutorial**: Intent includes "how to", step-by-step format needed
- **List post**: Keyword suggests enumeration ("best", "top", "types of")
- **Comparison**: Keywords include "vs", "alternative", "comparison"
- **Definition/explainer**: Simple informational, competitors average < 1500 words

**Target word count**: Beat the median of top 5 competitors by 20-30%, capped at reasonable length. Don't pad for length.

### Step 3 — Build the Outline

Generate a complete outline with every element specified.

## Output

Write to `docs/seo/<slug>/outline.md`:

```markdown
# Content Outline: {{Topic}}

**Date**: {{YYYY-MM-DD}}
**Content type**: {{Comprehensive guide / How-to / List / Comparison / Explainer}}

## SEO Metadata

| Element              | Value                                                       |
| -------------------- | ----------------------------------------------------------- |
| **Title tag**        | {{50-60 chars, primary keyword front-loaded}}               |
| **Meta description** | {{150-160 chars, includes CTA + value prop}}                |
| **URL slug**         | {{3-5 words, hyphenated, lowercase, no year}}               |
| **H1**               | {{Primary keyword + unique angle, distinct from title tag}} |

## Content Structure

### Introduction (~{{N}} words)

- Hook: {{Specific hook approach — stat, question, pain point}}
- **Primary keyword** appears in first 100 words
- Context: {{What problem this solves and for whom}}
- Promise: {{What the reader will learn/gain}}
- Internal link opportunity: {{Link to related pillar content}}

### H2: {{Secondary Keyword 1 — Section Title}} (~{{N}} words)

- Key points to cover:
  - {{Point 1 — cite specific fact from research brief}}
  - {{Point 2}}
  - {{Point 3}}
- **Secondary keyword**: {{exact keyword to include}}
- **LSI terms**: {{terms to weave in}}
- E-E-A-T signal: {{What experience/expertise signal to include}}
- Visual: {{Table / diagram / screenshot / code block suggested}}

### H2: {{Secondary Keyword 2 — Section Title}} (~{{N}} words)

...

### H2: {{Additional sections as needed}} (~{{N}} words)

...

### H2: FAQ (~{{N}} words)

Based on People Also Ask and forum questions:

#### H3: {{Question 1}}?

- Answer: {{40-60 words, snippet-optimized, direct answer first}}
- Source: {{Reference from research brief}}

#### H3: {{Question 2}}?

...

#### H3: {{Question 3}}?

...

(3-7 FAQ items)

### Conclusion (~{{N}} words)

- Recap key takeaways (3-5 bullet points)
- **Primary keyword** restated naturally
- CTA: {{Specific next action for the reader}}
- Internal link: {{Link to related content or product page}}

## Word Count

| Section      | Target    |
| ------------ | --------- |
| Introduction | {{N}}     |
| {{H2 1}}     | {{N}}     |
| {{H2 2}}     | {{N}}     |
| ...          | ...       |
| FAQ          | {{N}}     |
| Conclusion   | {{N}}     |
| **Total**    | **{{N}}** |

## Keyword Distribution Plan

| Keyword         | Type      | Placement                         |
| --------------- | --------- | --------------------------------- |
| {{primary}}     | Primary   | Title, H1, intro, conclusion, URL |
| {{secondary 1}} | Secondary | H2 heading, section body          |
| {{secondary 2}} | Secondary | H2 heading, section body          |
| ...             |           |                                   |
| {{LSI terms}}   | LSI       | Distributed naturally in body     |

## Internal Link Targets

| Anchor Text          | Target URL              | Location         |
| -------------------- | ----------------------- | ---------------- |
| {{descriptive text}} | {{/blog/related-post/}} | {{Section name}} |
| ...                  |                         |                  |

(3-5 internal links planned)

## External Link Targets

| Source                   | URL     | Location         |
| ------------------------ | ------- | ---------------- |
| {{Authoritative source}} | {{URL}} | {{Section name}} |

(1-2 external links planned)

## Schema Markup Plan

- [ ] **Article** schema (headline, author, dates, image)
- [ ] **FAQPage** schema (from FAQ section)
- [ ] **BreadcrumbList** schema (Home > Blog > {{Post Title}})
- [ ] **HowTo** schema (if tutorial format)

## E-E-A-T Signals Checklist

### Experience

- [ ] {{Specific experience signal: case study, screenshot, real example}}
- [ ] {{Another experience signal}}

### Expertise

- [ ] Author attribution: {{Name, title, credentials}}
- [ ] Technical depth: {{Specific depth indicator}}
- [ ] Methodology explained: {{Where/how}}

### Authoritativeness

- [ ] Internal links to related content (topical authority)
- [ ] Citations to authoritative sources

### Trustworthiness

- [ ] All statistics have cited sources
- [ ] Published date and last-updated date
- [ ] AI disclosure if applicable

## Image Plan

| Image                  | Alt Text                          | Location    |
| ---------------------- | --------------------------------- | ----------- |
| Featured image         | {{descriptive, keyword-relevant}} | Top         |
| {{Diagram/screenshot}} | {{descriptive}}                   | {{Section}} |

(Minimum 1 custom image)
```

## Quality Checks

Before saving, verify:

- [ ] Title tag is 50-60 characters with primary keyword front-loaded
- [ ] Meta description is 150-160 characters with CTA
- [ ] URL slug is 3-5 words, lowercase, hyphenated
- [ ] H1 is distinct from title tag but contains primary keyword
- [ ] Every H2 has a secondary keyword assigned
- [ ] Primary keyword appears in: title, H1, intro (first 100 words), conclusion, URL
- [ ] FAQ has 3-7 questions with 40-60 word answers
- [ ] Total word count beats competitor median by ~20-30%
- [ ] At least 3 internal links planned
- [ ] At least 1 external link to authoritative source
- [ ] Schema markup plan includes Article + BreadcrumbList minimum
- [ ] E-E-A-T checklist has specific, actionable items (not generic)
- [ ] Every key fact references the research brief source

## User Action

After generating, tell the user:

> Review the outline in `docs/seo/<slug>/outline.md`. Modify headings, reorder sections, adjust word count targets, or add/remove FAQs as needed. Once you're satisfied, run `/seo-write <slug>` to generate the full draft.
