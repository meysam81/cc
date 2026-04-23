---
name: seo-review
description: Audit a blog post draft against SEO best practices with a 100-point scoring rubric. Identifies specific issues and provides fix recommendations.
---

# SEO Blog Post Review & Audit

Score a blog post draft against a 100-point SEO rubric. Identifies specific issues per category and provides actionable fix recommendations.

## References

Read these for evaluation criteria (relative to this skill's base directory):

- `../../references/on-page-seo-checklist.md`
- `../../references/eeat-signals.md`
- `../../references/schema-markup-templates.md`

## Inputs

1. **Draft** — `docs/seo/<slug>/draft.md` (required)
2. **Outline** — `docs/seo/<slug>/outline.md` (required — used to verify structure compliance)
3. **Keyword data** — `docs/seo/<slug>/keyword-data.md` (required — used to verify keyword usage)
4. **Article directory** — user specifies `docs/seo/<slug>/` or skill finds the directory containing `draft.md`

## Article Directory Convention

All per-article files live in `docs/seo/<slug>/`. The user provides the slug, or the skill finds it by looking for the directory containing `draft.md`.

## Scoring Rubric (100 points)

### Category 1: Keyword Optimization (20 points)

| Check                               | Points | Pass Criteria                                       |
| ----------------------------------- | ------ | --------------------------------------------------- |
| Primary keyword in title tag        | 3      | Present in first 60 chars                           |
| Primary keyword in H1               | 3      | Present, naturally phrased                          |
| Primary keyword in first 100 words  | 3      | Present in opening paragraph                        |
| Primary keyword in meta description | 2      | Present in 150-160 char description                 |
| Primary keyword in URL slug         | 2      | Present in slug                                     |
| Secondary keywords in H2 headings   | 4      | Each H2 maps to a secondary keyword per outline     |
| LSI terms distributed               | 3      | At least 5 LSI terms from keyword data used in body |

**Scoring**: Each check is pass/fail for its point value. Partial credit if keyword is present but awkwardly placed.

### Category 2: Content Structure (20 points)

| Check                     | Points | Pass Criteria                                              |
| ------------------------- | ------ | ---------------------------------------------------------- |
| Word count target met     | 4      | Within ±10% of outline total                               |
| Heading hierarchy correct | 4      | Follows outline exactly, no skipped levels                 |
| FAQ section present       | 3      | 3-7 Q&A pairs with snippet-optimized answers               |
| Schema markup valid       | 3      | Article + BreadcrumbList minimum, valid JSON-LD            |
| Scannable formatting      | 3      | Bullets, bold terms, tables used; no wall-of-text sections |
| Section word counts       | 3      | Each section within ±15% of outline target                 |

### Category 3: E-E-A-T Signals (20 points)

| Check              | Points | Pass Criteria                                                 |
| ------------------ | ------ | ------------------------------------------------------------- |
| Author attribution | 4      | Name, title/credentials in frontmatter or body                |
| Experience signals | 5      | At least 2 concrete examples (case study, real data, process) |
| Expert citations   | 4      | At least 2 cited experts/sources with attribution             |
| Methodology shown  | 4      | Reasoning/framework explained, not just conclusions           |
| Trust signals      | 3      | Sources cited, dates present, factual claims verifiable       |

### Category 4: Readability (15 points)

| Check              | Points | Pass Criteria                                                     |
| ------------------ | ------ | ----------------------------------------------------------------- |
| Paragraph length   | 4      | No paragraph exceeds 4 sentences                                  |
| Sentence variety   | 3      | Mix of short (< 15 words) and medium (15-25 words) sentences      |
| Scannable elements | 3      | At least 3 different formats used (bullets, tables, bold, lists)  |
| Grade level        | 3      | Approximately grade 8-10 (accessible but not oversimplified)      |
| No filler phrases  | 2      | No "In today's world", "It goes without saying", "As we all know" |

### Category 5: On-Page SEO (15 points)

| Check                   | Points | Pass Criteria                                          |
| ----------------------- | ------ | ------------------------------------------------------ |
| Title tag length        | 2      | 50-60 characters                                       |
| Meta description length | 2      | 150-160 characters                                     |
| URL slug format         | 2      | 3-5 words, lowercase, hyphenated, no dates             |
| Image alt text          | 3      | Every image has descriptive, keyword-relevant alt text |
| Internal links          | 3      | 3-5 links with descriptive anchor text                 |
| External links          | 3      | 1-2 links to authoritative sources                     |

### Category 6: Brand Voice (10 points)

| Check               | Points | Pass Criteria                                                |
| ------------------- | ------ | ------------------------------------------------------------ |
| Tone match          | 4      | Matches brand voice document (professional/casual/technical) |
| Terminology correct | 3      | Uses brand-specific terms consistently                       |
| Audience alignment  | 3      | Complexity level matches target ICP                          |

## Process

### Step 1 — Read All Inputs

Read draft, outline, and keyword data files.

### Step 2 — Score Each Category

Evaluate every check in the rubric. For each:

- **Pass**: Award full points
- **Partial**: Award half points, note what's missing
- **Fail**: Award 0 points, provide specific fix

### Step 3 — Generate Report

## Output Format

Present the review directly in the conversation (do NOT write to a file):

```markdown
# SEO Review: {{Post Title}}

## Score: {{N}}/100 — {{PASS ✅ / NEEDS WORK ⚠️ / FAIL ❌}}

(Pass = 80+, Needs Work = 60-79, Fail = <60)

---

### Keyword Optimization: {{N}}/20

| Check                      | Score   | Status   | Note              |
| -------------------------- | ------- | -------- | ----------------- |
| Primary in title           | {{N}}/3 | ✅/⚠️/❌ | {{specific note}} |
| Primary in H1              | {{N}}/3 | ✅/⚠️/❌ | {{note}}          |
| Primary in first 100 words | {{N}}/3 | ✅/⚠️/❌ | {{note}}          |
| Primary in meta            | {{N}}/2 | ✅/⚠️/❌ | {{note}}          |
| Primary in URL             | {{N}}/2 | ✅/⚠️/❌ | {{note}}          |
| Secondaries in H2s         | {{N}}/4 | ✅/⚠️/❌ | {{note}}          |
| LSI distribution           | {{N}}/3 | ✅/⚠️/❌ | {{note}}          |

### Content Structure: {{N}}/20

(same table format)

### E-E-A-T Signals: {{N}}/20

(same table format)

### Readability: {{N}}/15

(same table format)

### On-Page SEO: {{N}}/15

(same table format)

### Brand Voice: {{N}}/10

(same table format)

---

## Priority Fixes

(List only items that lost points, ordered by point value — highest impact first)

### 1. {{Issue name}} ({{N}} points at stake)

**Problem**: {{What's wrong, with specific text quote from draft}}
**Fix**: {{Exact recommendation — what to change and where}}

### 2. {{Issue name}} ({{N}} points at stake)

...

---

## Strengths

- {{What the draft does well — be specific}}
- {{Another strength}}

## Verdict

{{If score >= 80}}: Ready to publish. Apply the minor fixes above for a perfect score.
{{If score 60-79}}: Needs revision. Apply the fixes above and re-run `/seo-review`.
{{If score < 60}}: Significant rework needed. Consider re-running `/seo-write` with updated outline.
```

## Evaluation Guidelines

**Be strict but fair:**

- Don't penalize style choices that are valid but different from your preference
- DO penalize missing SEO fundamentals (keyword placement, heading structure)
- Award partial credit generously when the intent is right but execution is imperfect
- Quote specific text from the draft when noting issues (makes fixes actionable)

**Common issues to watch for:**

- Keyword stuffing (density > 2%)
- Wall-of-text paragraphs (> 4 sentences)
- Missing H2 ↔ secondary keyword mappings
- FAQ answers that are too long (> 80 words) or too short (< 30 words)
- Generic E-E-A-T signals ("experts say...") without specific attribution
- Broken heading hierarchy (H1 → H3 without H2)
- Filler introductions that delay value delivery

## User Action

After the review, tell the user:

> If score >= 80: "The draft is ready. Apply the minor fixes noted above, then publish or run `/seo-optimize` after publishing to verify on-page performance."
> If score < 80: "Apply the fixes above, then re-run `/seo-review <slug>` to check your score. Alternatively, re-run `/seo-write <slug>` if major structural changes are needed."
