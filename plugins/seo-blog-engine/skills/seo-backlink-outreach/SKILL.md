---
name: seo-backlink-outreach
description: Analyze site pages, discover backlink prospects via SERP analysis and web search, and generate A/B outreach email templates with spintax and follow-up sequences.
tools:
  - mcp__dataforseo__serp_organic_live_advanced
  - mcp__dataforseo__backlinks_summary
  - mcp__dataforseo__content_analysis_search
  - mcp__dataforseo__dataforseo_labs_google_ranked_keywords
  - mcp__dataforseo__dataforseo_labs_google_serp_competitors
  - mcp__dataforseo__dataforseo_labs_google_relevant_pages
  - mcp__dataforseo__dataforseo_labs_bulk_traffic_estimation
  - mcp__dataforseo__ai_opt_kw_data_loc_and_lang
---

# Backlink Outreach Strategy Generator

Analyze site content, discover high-value backlink prospects, and generate personalized outreach email templates. Operates in two modes: single-page (target one slug) or full-site audit (analyze all pages and prioritize).

## References

Read before starting (relative to this skill's base directory):

- `../../references/backlink-outreach-guide.md`
- `../../references/dataforseo-workflows.md`
- `../../references/eeat-signals.md`

## Inputs

1. **Mode** — user specifies one of:
   - `/seo-backlink-outreach <slug>` — single page mode
   - `/seo-backlink-outreach --full-site` — analyze all pages, prioritize best candidates
   - `/seo-backlink-outreach` (no args) — default to full-site mode
2. **Content directory** — user-provided path → `./landing-page/` (if present) → CWD
3. **Brand voice** — `docs/seo/brand-voice.md` (required — need company name, domain, and niche for outreach)
4. **Topical clusters** — `docs/seo/topical-clusters.md` (optional)
5. **Existing keyword data** — `docs/seo/<slug>/keyword-data.md` (optional, reuse when available)
6. **Target market** — default US (location_code: 2840, language_code: "en"); ask if different

If `brand-voice.md` is missing, ask the user to create it first — outreach cannot be personalized without knowing the brand name and domain.

## Browser Capabilities

Check the project's `CLAUDE.md` (or `.claude/CLAUDE.md`) for mentions of:

- **Browserless** (typically `localhost:30000`) — headless Chrome for JS-heavy sites
- **Splash** — alternative JS rendering proxy

If available, use these for prospect qualification when standard web fetches are blocked by anti-bot measures or when pages require JavaScript rendering. This expands the skill's reach against sites with aggressive bot detection.

## Process

### Phase 1 — Content Exploration (MANDATORY — NON-NEGOTIABLE)

This phase MUST be thorough. Skimming is not acceptable. The quality of prospect discovery depends entirely on how well we understand the content.

#### Content Discovery

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

#### Dispatch Exploration Subagents

**Dispatch exploration subagent(s) via the Task tool.**

For sites with **≤ 20 content pages**: dispatch a single Explore subagent.
For sites with **> 20 content pages**: split by directory, cluster, or content type — one subagent per group.

Each subagent extracts per page:

| Field              | Source                                               |
| ------------------ | ---------------------------------------------------- |
| File path          | File system path                                     |
| URL slug           | Derived from file path and framework routing         |
| Title              | H1 or frontmatter `title`                            |
| Primary keyword    | From frontmatter, H1, or inferred                    |
| Content summary    | 2-3 sentence description of what the page covers     |
| Search intent      | Informational / Commercial / Transactional           |
| Content depth      | Thin (< 800w) / Standard (800-2000w) / Deep (2000w+) |
| Unique angle       | What makes this page different from competitors      |
| Linkable assets    | Original data, tools, infographics, templates        |
| Existing backlinks | From `keyword-data.md` if available                  |
| Word count         | Approximate                                          |

### Phase 2 — Backlink Worthiness Assessment

Rate each page's backlink potential (do NOT delegate this):

| Criterion              | Weight | Score 1-5                                              |
| ---------------------- | ------ | ------------------------------------------------------ |
| Content depth          | High   | 5 = comprehensive deep-dive, 1 = thin/shallow          |
| Unique data/research   | High   | 5 = original research/data, 1 = nothing original       |
| Linkable assets        | High   | 5 = tools/calculators/infographics, 1 = text-only      |
| Fresh angle            | Medium | 5 = contrarian/novel take, 1 = same as everyone else   |
| Search intent value    | Medium | 5 = high commercial value, 1 = low-value informational |
| Current backlink count | Medium | 5 = few/no backlinks (opportunity), 1 = already strong |

**Priority score** = weighted average. Pages scoring < 2.5 overall are skipped for outreach (note in report as "not recommended for outreach — reason: {{reason}}").

In **single-page mode**, skip this scoring — the user already chose the page.

In **full-site mode**, present the priority matrix and select the top 3-5 candidates for prospect discovery.

### Phase 3 — Prospect Discovery

For each target page (top candidates from Phase 2, or the single specified page):

#### Step 3a — SERP Analysis (DataForSEO)

Use `serp_organic_live_advanced` for the page's primary keyword:

```
keyword: "{{primary keyword}}"
location_code: 2840
language_code: "en"
device: "desktop"
depth: 10
```

From top 10 results, identify:

- **Listicles**: titles containing "best", "top", numbers, "tools", "resources"
- **Resource pages**: titles containing "guide", "resources", "links", "directory"
- **How-to/informational**: pages that mention our topic but link elsewhere for depth
- **Roundup posts**: weekly/monthly compilations in our niche

#### Step 3b — Competitor Backlink Analysis (DataForSEO)

Use `backlinks_summary` on the top 3-5 SERP results:

```
target: "{{competitor URL}}"
```

Identify domains linking to competitors but not to us — these are warm prospects.

Use `serp_competitors` to find additional competing domains:

```
keywords: ["{{primary keyword}}", "{{secondary keywords}}"]
location_code: 2840
```

#### Step 3c — Web Mentions (DataForSEO)

Use `content_analysis_search` to find pages mentioning our brand or topic:

```
keyword: "{{brand name OR topic}}"
search_mode: "as_is"
```

Pages mentioning our brand without linking = easy wins (ask for link attribution).

#### Step 3d — Prospect Qualification (WebSearch / Firecrawl)

For each candidate prospect (aim for 10-15 per target page):

1. **WebSearch** for: `site:{{prospect-domain}} about` or `site:{{prospect-domain}} contact` to find about/contact pages
2. **Fetch the prospect's page** to confirm:
   - Content is still live and relevant
   - There's a natural placement for our link (specific section, list, or context)
   - The site looks legitimate (not a link farm or scraper)
3. **Extract contact info** (if findable): author name, email, social profiles
4. **Note personalization hooks**: specific quotes, positions, or insights from their content that show we actually read it

If a site blocks standard web fetches and browserless/splash is available per CLAUDE.md, use that for rendering.

#### Step 3e — Existing Data Reuse

If `docs/seo/<slug>/keyword-data.md` exists for the target page:

- Reuse competitor analysis data (don't re-fetch what's already known)
- Supplement with fresh prospect-specific searches

### Phase 4 — Outreach Template Generation

For each qualified prospect, generate:

#### Template A — Casual / Peer

```
Subject: {Quick question|Thought on|Regarding} your {{topic}} {article|post|guide}

{Hi|Hey|Hello} {{first_name}},

{I came across|I was reading|I found} your {article|piece|post} on {{their_article_topic}} — {{personalized_compliment_about_specific_detail}}.

{I noticed|It occurred to me that|I thought} our {{content_description}} on {{our_topic}} might {complement|add to|be a useful addition to} your {{specific_section_or_list}}. {{one_sentence_on_what_our_content_adds}}.

{Would you consider|Do you think it'd make sense to|Happy to share if you think it'd be worth} {adding a link|including it|mentioning it}?

{Cheers|Best|Thanks},
{{sender_name}}
{{sender_title}}, {{company}}
```

#### Template B — Professional / Formal

```
Subject: Resource suggestion for your {{topic}} {article|guide|page}

{Hi|Hello} {{first_name}},

{{personalized_intro_referencing_their_specific_work_or_achievement}}.

We recently published {{content_title}} ({{URL}}) which covers {{specific_angle_or_data_point}}. {Based on|Given} what you covered in your {{their_section_name}}, {I believe|I think} our {research|guide|data} could {provide additional depth|offer a complementary perspective|give your readers more detail} on {{specific_subtopic}}.

{Would you be open to|Would it make sense to} including {a reference|a link} in your {{specific_section}}? {Happy to|I'm also glad to} return the favor if there's content of yours that fits our coverage.

{Best regards|Kind regards},
{{sender_name}}
{{sender_title}} at {{company}}
```

#### Follow-Up Sequence (3 touchpoints per template)

**Follow-up 1 (Day 3-4)**:

```
Subject: Re: {{original_subject}}

{Hi|Hey} {{first_name}},

{Just floating this back up|Wanted to quickly follow up} — {I know inboxes get busy|I know you're probably swamped}.

{One thing I didn't mention|Also worth noting}: {{new_angle_or_data_point_not_in_original}}.

{No rush at all|Happy to discuss if you're interested} — {just thought it might be useful|let me know what you think}.

{{sender_name}}
```

**Follow-up 2 (Day 7-8)**:

```
Subject: Re: {{original_subject}}

{Hi|Hey} {{first_name}},

{Came across|Found} {{related_resource_or_stat}} that {relates to|ties into} your {{their_article_topic}} — thought you might find it {interesting|useful} regardless of our previous note.

{{one_sentence_sharing_the_insight}}

{Cheers|Best},
{{sender_name}}
```

**Follow-up 3 (Day 14 — final)**:

```
Subject: Re: {{original_subject}}

{Hi|Hey} {{first_name}},

{Last note from me on this|Just a final check-in} — {completely understand if it's not a fit|no worries at all if the timing isn't right}.

If you ever {update that piece|revisit that article} and {think our resource could add value|want to add more resources}, {the offer stands|happy to chat}.

{All the best|Cheers},
{{sender_name}}
```

### Phase 5 — Generate Output

#### Full-Site Mode

Write `docs/seo/backlink-outreach/strategy-overview.md`:

```markdown
# Backlink Outreach Strategy

**Date**: {{YYYY-MM-DD}}
**Site**: {{domain/project}}
**Pages analyzed**: {{N}}
**Pages recommended for outreach**: {{N}}
**Total prospects identified**: {{N}}

## Priority Matrix

| #   | Page     | Primary Keyword | Depth Score | Unique Assets   | Current Backlinks | Priority  |
| --- | -------- | --------------- | ----------- | --------------- | ----------------- | --------- |
| 1   | {{slug}} | {{keyword}}     | {{1-5}}     | {{description}} | {{N}}             | 🔴 High   |
| 2   | ...      |                 |             |                 |                   | 🟡 Medium |
| ... |          |                 |             |                 |                   |           |

## Pages Not Recommended

| Page     | Reason                                                            |
| -------- | ----------------------------------------------------------------- |
| {{slug}} | {{Thin content / No unique angle / Already has strong backlinks}} |
| ...      |                                                                   |

## Campaign Metrics to Track

| Metric                       | Baseline | Target (3 months)        |
| ---------------------------- | -------- | ------------------------ |
| Total referring domains      | {{N}}    | {{N + realistic growth}} |
| Emails to send               | —        | {{N}}                    |
| Expected responses (10-15%)  | —        | {{N}}                    |
| Expected links earned (3-5%) | —        | {{N}}                    |

## Next Steps

1. Review individual outreach files in `docs/seo/backlink-outreach/`
2. Customize email templates with your sender details
3. Load into your outreach tool (Lemlist, Woodpecker, etc.)
4. Track responses and update this strategy monthly
```

#### Per-Page Outreach Files

Write `docs/seo/backlink-outreach/{{slug}}-outreach.md` for each target page:

```markdown
# Backlink Outreach: {{Page Title}}

**Target page**: {{slug}} ({{URL if known}})
**Primary keyword**: {{keyword}}
**Content summary**: {{2-3 sentences}}
**Unique selling point**: {{What makes this page link-worthy}}

## Prospects

### Prospect 1: {{Site Name}}

| Field                | Value                                                           |
| -------------------- | --------------------------------------------------------------- |
| URL                  | {{prospect page URL}}                                           |
| Type                 | {{Listicle / Resource page / How-to / Roundup / Brand mention}} |
| Contact              | {{Name, email if found}}                                        |
| Relevance            | {{High/Medium}}                                                 |
| Outreach angle       | {{Why they'd include us — specific section or context}}         |
| Personalization hook | {{Specific detail from their content}}                          |

### Prospect 2: {{Site Name}}

...

(10-15 prospects per page)

## Email Templates

### Template A — Casual

**Initial outreach**:
{{Full Template A with spintax, personalized per this page's context}}

**Follow-up 1 (Day 3-4)**:
{{Casual follow-up}}

**Follow-up 2 (Day 7-8)**:
{{Value-add follow-up}}

**Follow-up 3 (Day 14)**:
{{Graceful close}}

### Template B — Professional

**Initial outreach**:
{{Full Template B with spintax, personalized per this page's context}}

**Follow-up 1 (Day 3-4)**:
{{Professional follow-up}}

**Follow-up 2 (Day 7-8)**:
{{Value-add follow-up}}

**Follow-up 3 (Day 14)**:
{{Graceful close}}

## Per-Prospect Customization Notes

| Prospect | Personalized Intro | Specific Section to Reference | New Angle for Follow-up   |
| -------- | ------------------ | ----------------------------- | ------------------------- |
| {{name}} | {{custom intro}}   | {{their section name}}        | {{data point or insight}} |
| ...      |                    |                               |                           |
```

## Quality Checks

Before saving output, verify:

- [ ] Every email template is 3-5 sentences max (excluding subject line)
- [ ] No spam trigger words: "free", "exclusive", "guaranteed", "act now", "limited time"
- [ ] Every prospect has a specific outreach angle (not generic "we wrote something similar")
- [ ] Spintax is syntactically correct: `{variant1|variant2|variant3}`
- [ ] Each follow-up adds something new (not just "bumping this")
- [ ] Personalization hooks reference specific details from prospect's content
- [ ] Pages scoring < 2.5 on worthiness are excluded with documented reason
- [ ] Templates A and B are genuinely different in tone (not just word substitutions)
- [ ] Contact information is real (not fabricated or guessed)
- [ ] All DataForSEO data is from current run (not stale)

## User Action

After generating, tell the user:

> Review your outreach strategy in `docs/seo/backlink-outreach/`. The strategy overview shows which pages to prioritize. Each page's outreach file has 10-15 qualified prospects with personalized email templates in A/B variants plus 3 follow-ups each. Customize the sender details, load into your outreach tool, and start sending. Track responses and re-run monthly to discover new prospects.
