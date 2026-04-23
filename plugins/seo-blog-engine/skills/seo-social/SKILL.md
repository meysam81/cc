---
name: seo-social
description: Generate platform-specific social media posts from a blog draft. Covers X, Bluesky, Mastodon, LinkedIn, Reddit, Patreon, and Buy Me a Coffee in 3 tiers (short-form, long-form, patronage).
---

# Social Media Post Generator

Generate platform-specific social media content from a published or ready-to-publish blog post. Produces a single file with posts tailored for 7 platforms across 3 tiers.

## References

Read this before generating (relative to this skill's base directory):

- `../../references/social-media-platforms.md`

## Inputs

1. **Draft** — `docs/seo/<slug>/draft.md` (required — the blog post)
2. **Outline** — `docs/seo/<slug>/outline.md` (optional — for primary keyword and keyword data)
3. **Brand voice** — `docs/seo/brand-voice.md` (optional — for tone adaptation)
4. **Article directory** — user specifies `docs/seo/<slug>/` or skill finds the directory containing `draft.md`

## Article Directory Convention

All per-article files live in `docs/seo/<slug>/`. The user provides the slug, or the skill finds it by looking for the directory containing `draft.md`.

## Mode Selection

**Default mode (3-tier):** Generates one post per platform using shared insight extraction per tier, then applies platform-specific formatting and tone adjustments.

**Per-platform mode:** If the user provides `--per-platform` as an argument, generate each of the 7 posts independently with full platform-specific voice, tone, and cultural adaptation. This uses more tokens but produces maximum authenticity per platform.

## Process

### Step 1 — Read and Analyze the Draft

Read `draft.md` completely. Also read `outline.md` and `brand-voice.md` if they exist.

Extract from the draft:

- **Blog title** (from H1 or frontmatter `title`)
- **Primary keyword** (from frontmatter `primary_keyword` or outline)
- **Blog URL/slug** (from frontmatter `slug`)
- **Core insight** — the single most compelling takeaway the reader walks away with
- **Key supporting points** — 3-5 specific facts, stats, or examples that reinforce the core insight
- **Target audience** — inferred from brand voice or draft content
- **Unique angle** — what makes this post different from competitors covering the same topic

### Step 2 — Generate Short-Form Posts (Tier 1)

Generate posts for X, Bluesky, and Mastodon. Each must be a genuinely different take on the core insight, not the same text truncated to different lengths.

**X (Twitter) — 280 chars max:**

- Lead with a punchy, opinionated hook
- Include the blog link naturally
- Add 1-2 relevant hashtags
- Personality-driven; avoid corporate tone
- Aim for 70-100 chars of text before the link for maximum impact

**Bluesky — 300 chars max:**

- Conversational, authentic tone
- Include the blog link (renders as a link card with preview)
- Minimal or no hashtags — discovery happens via custom feeds
- Anti-corporate voice; write like a person, not a brand

**Mastodon — 500 chars max:**

- Slightly more detail than X/Bluesky; use the extra space for context
- 3-5 CamelCase hashtags (e.g., #WebDevelopment, not #webdevelopment)
- Suggest a Content Warning (CW) line if the topic could be sensitive
- Community-first tone; inclusive language
- Include the blog link; Mastodon is link-friendly

### Step 3 — Generate Long-Form Posts (Tier 2)

Generate posts for LinkedIn and Reddit. These are self-contained — a reader should get value without clicking the link.

**LinkedIn — 1,300-1,900 chars:**

- **Hook (first 210 chars)**: Must be compelling and self-contained. This is what appears before "See more" on desktop. Write it as if it's the only thing people will read.
- **Body**: Professional insight framing. Use storytelling or "here's what I learned" structure. Short paragraphs (1-2 sentences). White space between paragraphs.
- **Link placement**: At the very end, after delivering full value. LinkedIn penalizes link-first posts ~60% in reach.
- **Hashtags**: 3-5 relevant hashtags at the bottom
- **Primary keyword**: Include naturally for LinkedIn SEO
- **Do NOT**: Use emojis as bullet points. No "🔥 Hot take:" or "💡 Insight:" patterns.

**Reddit:**

- **Framing**: "Lessons learned" / "Here's how we solved X" / "I built X, here's what happened"
- **Tone**: Value-first, anti-marketing, authentic. Write like you're sharing with peers, not promoting
- **Structure**: Use markdown headers, bullets, and code blocks for readability
- **Link**: At the bottom: "Full write-up with more details: [link]"
- **No hashtags, minimal emojis**
- **Suggested subreddits**: Include 2-4 relevant subreddit suggestions based on the blog topic
- **Note**: Add reminder about checking subreddit rules and the 90/10 rule

### Step 4 — Generate Patronage Posts (Tier 3)

Generate teasers for Patreon and Buy Me a Coffee. These are NOT excerpts — they're framed as exclusive/extended content teasers that drive subscriptions.

**Patreon — ~300 words:**

- Behind-the-scenes framing: share the story behind the blog post
- Tease extended content that patrons get: deeper analysis, source code, templates, raw data, video walkthrough, etc.
- Suggest which membership tier this content fits
- Suggest an "Extras" idea (a digital product tie-in if applicable)
- Personal, intimate tone — patrons are paying for access to you, not just content

**Buy Me a Coffee — ~150 words:**

- Gratitude-focused: "If you found [blog title] helpful..."
- Brief, warm, simple
- Mention what their support enables (more posts, tools, research)
- Clear but soft CTA
- Do not hard-sell; this is a tip jar, not a subscription pitch

### Step 5 — Quality Checks

Before outputting, verify every item:

**Character limits:**

- [ ] X post ≤ 280 chars
- [ ] Bluesky post ≤ 300 chars
- [ ] Mastodon post ≤ 500 chars
- [ ] LinkedIn post 1,300-1,900 chars (within 3,000 max)

**Content quality:**

- [ ] LinkedIn hook (first 210 chars) works as a standalone teaser
- [ ] Reddit post reads as value-first, not promotional
- [ ] Mastodon hashtags are CamelCase
- [ ] Primary keyword appears naturally in LinkedIn and Reddit posts
- [ ] No copy-paste between platforms — each is genuinely adapted to platform culture
- [ ] Brand voice is consistent across all tiers (adapted per platform but recognizably the same author)

**Patronage quality:**

- [ ] Patreon post teases exclusive content, not just repeats the blog
- [ ] Buy Me a Coffee post is gratitude-focused, not sales-focused
- [ ] Both patronage posts have clear but non-aggressive CTAs

**If any check fails:** Fix the post before outputting. Recount characters if edits were made.

## Output

Write to `docs/seo/<slug>/social-posts.md`:

```
---
title: "Social Posts: {{blog title}}"
source: "docs/seo/<slug>/draft.md"
generated: "{{YYYY-MM-DD}}"
primary_keyword: "{{primary keyword}}"
---

# Social Media Posts

## Short-Form

### X (Twitter)
> {{post text including link}}

**Char count:** {{N}}/280
**Hashtags:** {{#tag1 #tag2}}

### Bluesky
> {{post text including link}}

**Char count:** {{N}}/300

### Mastodon
> {{post text including link}}

**Char count:** {{N}}/500
**CW (optional):** {{topic if applicable, or "None"}}
**Hashtags:** {{#CamelCaseTag1 #CamelCaseTag2 #CamelCaseTag3}}

---

## Long-Form

### LinkedIn
> {{full post text with link at end}}

**Char count:** {{N}}/3,000
**Hook (first 210 chars):** {{preview of what appears before "See more"}}
**Hashtags:** {{#tag1 #tag2 #tag3}}

### Reddit
> {{full post text with link at bottom}}

**Suggested subreddits:** {{r/sub1, r/sub2, r/sub3}}
**Note:** Check each subreddit's rules before posting. Contribute value in comments before self-promoting.

---

## Patronage

### Patreon
> {{~300 word teaser}}

**Tier suggestion:** {{recommended tier}}
**Extras idea:** {{digital product tie-in or "N/A"}}

### Buy Me a Coffee
> {{~150 word update}}

---

## Platform-Specific Notes
- LinkedIn: Post is self-contained; link at end only. Algorithm penalizes link-first posts ~60%.
- X: Link penalty removed Oct 2025. Thread version recommended for deeper engagement.
- Reddit: Rewrite framing per subreddit culture. 90/10 rule applies.
- Bluesky: No link penalty. Consider thread (3-5 posts) for deeper engagement.
- Mastodon: CamelCase hashtags are a cultural norm. Alt text required for any images.
- Patreon: This is a teaser for exclusive content, not a blog promotion.
- Buy Me a Coffee: Keep it simple and grateful. Embed the widget on your blog.
```

## User Action

After generating, tell the user:

> Social posts saved to `docs/seo/<slug>/social-posts.md`. Review each platform's post and adjust for your personal voice before publishing. Pay special attention to:
>
> - LinkedIn hook (first 210 chars) — this determines 60-70% of engagement
> - Reddit framing — adapt to each subreddit's specific culture
> - Character counts — verify before posting, especially X and Bluesky
>
> For deeper per-platform customization, re-run with `/seo-social <slug> --per-platform`.
