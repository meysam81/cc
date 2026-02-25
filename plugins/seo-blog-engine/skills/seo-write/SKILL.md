---
name: seo-write
description: Write a complete SEO-optimized blog post from an approved outline, research brief, and brand voice. Produces a publish-ready draft with schema markup.
model: opus
---

# SEO Blog Post Writer

Write a complete, publish-ready blog post from an approved outline. The draft follows the outline exactly, integrates research data, matches brand voice, and meets all on-page SEO requirements.

## References

Read these before writing (relative to this skill's base directory):

- `../../references/on-page-seo-checklist.md`
- `../../references/eeat-signals.md`
- `../../references/schema-markup-templates.md`

## Inputs

The first three are required:

1. **Approved outline** — `docs/seo/<slug>/outline.md` (from `seo-outline`, user-reviewed)
2. **Research brief** — `docs/seo/<slug>/research-brief.md`
3. **Brand voice** — `docs/seo/brand-voice.md`
4. **Topical clusters** — `docs/seo/topical-clusters.md` (if exists) — use to weave in internal links to sibling/related posts
5. **Article directory** — user specifies `docs/seo/<slug>/` or skill finds the directory containing `outline.md` and `research-brief.md`

## Article Directory Convention

All per-article files live in `docs/seo/<slug>/`. The user provides the slug, or the skill finds it by looking for the directory containing the required input files.

## Process

### Step 1 — Internalize All Inputs

Read all three files completely. Before writing, confirm understanding of:

- Brand tone and terminology (from brand voice)
- Heading structure and word count targets (from outline)
- Key facts, stats, quotes to integrate (from research brief)
- Keyword distribution plan (from outline)
- E-E-A-T signals to embed (from outline)

### Step 2 — Write the Full Draft

Write the complete post in one pass, following the outline structure exactly.

**Writing rules:**

**Keyword optimization:**

- Primary keyword in first 100 words (naturally, not forced)
- Primary keyword in H1 (from outline)
- Primary keyword in conclusion paragraph
- Secondary keywords in their assigned H2 headings
- LSI terms distributed naturally throughout body
- Keyword density: 1-2% maximum — if it reads awkwardly, reduce
- Never bold or italicize keywords just for SEO

**Structure:**

- Follow outline heading hierarchy exactly (H1, H2, H3)
- Hit word count targets per section (±10% is acceptable)
- Short paragraphs: 2-4 sentences maximum
- Use varied formatting: bullets, numbered lists, bold key terms, tables
- Transition sentences between every section
- One idea per paragraph

**E-E-A-T signals:**

- Integrate experience examples where the outline specifies
- Cite sources inline: "According to [Source] (Year)..." or "[Stat] ([Source])"
- Include methodology/reasoning behind recommendations
- Author perspective: use "we" for organizational experience, specific examples

**FAQ section:**

- Write answers in snippet-optimized format
- Lead with direct answer (first sentence answers the question completely)
- Expand with 1-2 supporting sentences
- Target 40-60 words per answer
- Match question text exactly to H3 headings

**Links:**

- Internal links at first natural mention of related topic
- Use descriptive anchor text (not "click here" or "read more")
- External links to authoritative sources as specified in outline
- Every factual claim backed by research brief citation

**Images:**

- Where outline specifies images, write descriptive alt text suggestions
- Format: `![Alt text description](image-placeholder.webp)`
- Include caption suggestions where helpful

**Tone:**

- Match brand voice document exactly
- Human, not robotic — vary sentence length and structure
- Conversational where appropriate, technical where needed
- No filler phrases ("In today's world...", "It goes without saying...")
- No generic introductions — start with the hook specified in outline

### Step 3 — Generate Schema Markup

Based on the outline's schema plan, generate complete JSON-LD blocks using the templates from `schema-markup-templates.md`.

- Always include Article schema
- Always include BreadcrumbList schema
- Include FAQPage schema if FAQ section exists
- Include HowTo schema if tutorial format
- Use placeholder values that are easy to fill in: `{{AUTHOR_NAME}}`, `{{PUBLISH_DATE}}`, etc.

### Step 4 — Self-Review Checklist

Before outputting, verify every item:

**Keyword optimization:**

- [ ] Primary keyword in first 100 words
- [ ] Primary keyword in H1
- [ ] Primary keyword in conclusion
- [ ] Secondary keywords in assigned H2 headings
- [ ] LSI terms distributed (not clustered)
- [ ] No keyword stuffing (reads naturally)

**Structure:**

- [ ] All outline headings present in correct order
- [ ] Word count targets met (±10%)
- [ ] Paragraphs are 2-4 sentences
- [ ] Bullets/lists/tables used for scannability
- [ ] Smooth transitions between sections

**E-E-A-T:**

- [ ] Experience signals embedded (case studies, examples)
- [ ] Expert citations included with sources
- [ ] Methodology/reasoning explained
- [ ] All facts traceable to research brief

**On-page SEO:**

- [ ] Internal links placed (3-5)
- [ ] External links placed (1-2)
- [ ] Image alt text suggestions included
- [ ] FAQ answers are 40-60 words, snippet-optimized

**Brand voice:**

- [ ] Tone matches brand voice document
- [ ] Correct terminology used throughout
- [ ] Audience-appropriate complexity level

## Output

Write the complete draft to `docs/seo/<slug>/draft.md`:

```markdown
---
title: "{{Title tag from outline — 50-60 chars}}"
meta_description: "{{Meta description from outline — 150-160 chars}}"
slug: "{{URL slug from outline}}"
primary_keyword: "{{primary keyword}}"
author: "{{AUTHOR_NAME}}"
date: "{{YYYY-MM-DD}}"
---

# {{H1 from outline}}

{{Full blog post content following outline structure...}}

---

## Schema Markup

{{JSON-LD blocks}}
```

## Quality Standards

The draft must meet these minimum standards:

| Criterion        | Requirement                                       |
| ---------------- | ------------------------------------------------- |
| Keyword density  | 1-2% for primary keyword                          |
| Paragraph length | 2-4 sentences max                                 |
| Readability      | Grade level 8-10 (accessible but not dumbed down) |
| Facts cited      | Every statistic has a source reference            |
| Internal links   | 3-5 minimum                                       |
| External links   | 1-2 minimum                                       |
| FAQ answers      | 40-60 words each                                  |
| Schema markup    | Article + BreadcrumbList minimum                  |

## User Action

After generating, tell the user:

> Draft saved to `docs/seo/<slug>/draft.md`. Review the content, then run `/seo-review <slug>` to get a detailed SEO quality score and specific improvement recommendations.
