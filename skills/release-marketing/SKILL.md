---
name: release-marketing
description: Generate marketing content (blog posts, landing pages, social media, changelogs) from recent git changes and product updates. Use when the user wants to create marketing materials after a release, PR merge, or deployment.
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash, Write, Edit, Agent
argument-hint: [blog|landing-page|social|changelog|all]
---

# Release Marketing Content Generator

You generate professional marketing content based on what was actually shipped in the product. Your output is always grounded in real code changes — never fabricated features.

## Context: What Just Shipped

Gather the raw material first. Run these to understand what changed:

```bash
# Recent changes since last tag or PR merge
!git log --oneline --no-merges -30
```

```bash
# Last merged PR
!gh pr list --state merged --limit 1 --json title,body,number,mergedAt
```

```bash
# Current branch
!git branch --show-current
```

## Step 1: Analyze Changes

Before generating ANY content, thoroughly understand what shipped:

1. **Identify the PR/release boundary**: Find the last merged PR or tag
2. **Read the diff**: `git diff <last-release>..HEAD` or `git diff <base>..HEAD`
3. **Categorize changes** into:
   - **User-facing features** (new UI, new workflows, new capabilities)
   - **Integrations** (CRM, API, third-party connections)
   - **Performance/UX improvements** (speed, layout, accessibility)
   - **Security enhancements** (auth, encryption, compliance)
   - **Infrastructure** (deployment, monitoring, reliability)
4. **Read the actual component files** that changed to understand the user experience — don't just summarize commit messages

Present the analysis to the user and confirm before generating content.

## Step 2: Determine Content Type

Based on the `$ARGUMENTS` (or ask if not specified):

| Argument | Output |
|----------|--------|
| `blog` | SEO-optimized blog post (HTML, matching existing page patterns) |
| `landing-page` | Industry or feature landing page (HTML) |
| `social` | LinkedIn post + Twitter/X thread + email snippet |
| `changelog` | User-facing changelog entry |
| `all` | All of the above |

If no argument, ask: "What content do you need? (blog / landing-page / social / changelog / all)"

## Step 3: Generate Content

### Brand Voice & Tone

Read the existing marketing pages in `public/` to match the established voice. Key principles:

- **Confident but not arrogant** — "finally delivers" not "the best ever"
- **Problem-first framing** — Lead with the pain point, then the solution
- **Specific numbers** — "2-minute prospect intelligence" not "fast research"
- **B2B sales language** — discovery, pipeline, deal velocity, close rates, MEDDIC
- **The 3 Unsolvable Problems framework**: (1) Prospect intelligence at scale, (2) Real-time discovery coaching, (3) Instant proposal generation
- **Active voice, direct** — No passive constructions, no filler
- **Matt Oess as thought leader** — Yale MBA, TechCXO partner, 20+ years enterprise sales

### Brand Identity

- **Product name**: Revenue Growth Agent (RGA)
- **Company**: TechCXO
- **Domain**: revenuegrowthagent.com
- **Colors**: techcxo-orange (#fe6601), techcxo-navy (#1B365D), techcxo-btorange (#fea735)
- **Logo**: /logo.png, /Blue-Rocket.svg, /Orange-Rocket.svg
- **Favicon**: /FAVICON40x40.png
- **GA ID**: G-F1BFC450CK

### For Blog Posts (HTML)

Read an existing blog page (e.g., `public/how-ai-sales-transformation-finally-works-for-b2b-teams.html`) and match:
- Full HTML document with `<!DOCTYPE html>`
- Enhanced meta tags (title, description, canonical URL)
- Open Graph tags (og:type, og:title, og:description, og:url, og:site_name, og:image)
- Twitter Card tags
- Schema.org structured data (`@graph` with Organization + WebPage or Article)
- Tailwind CSS via `/css/about-styles.min.css`
- Google Analytics snippet (G-F1BFC450CK)
- Custom dropdown styles for nav
- Consistent nav header and footer matching existing pages
- Responsive design (mobile-first Tailwind classes)

**SEO requirements:**
- Title tag under 60 characters
- Meta description 150-160 characters
- Canonical URL with trailing slash
- H1 contains primary keyword
- Schema.org Article markup for blog posts
- Internal links to other RGA pages where relevant

### For Landing Pages (HTML)

Same HTML structure as blog posts, plus:
- Schema.org HowTo or FAQPage markup where applicable
- Clear CTA buttons linking to `/?signup=true`
- Feature comparison sections
- Social proof / testimonials section
- FAQ section with Schema.org FAQPage markup

### For Social Media

**LinkedIn post format:**
- Hook line (pattern interrupt or bold claim)
- 3-5 short paragraphs with line breaks
- Specific results or metrics
- End with CTA or question for engagement
- 3-5 relevant hashtags
- Tag @TechCXO and @Revenue Growth Agent where appropriate

**Twitter/X thread format:**
- Tweet 1: Hook + key claim (under 280 chars)
- Tweets 2-4: Supporting points with specifics
- Final tweet: CTA with link
- Each tweet standalone-readable

**Email snippet:**
- Subject line (under 50 chars, curiosity-driven)
- Preview text (under 90 chars)
- 3-paragraph body: Problem → Solution → CTA

### For Changelog

- Date + version heading
- Grouped by: New Features, Improvements, Bug Fixes, Security
- Each entry: one-line summary + optional detail paragraph
- User-facing language (not technical jargon)
- Link to relevant docs or pages where applicable

## Step 4: File Output

Write generated content to appropriate locations:

| Type | Output Path |
|------|-------------|
| Blog post | `public/{seo-friendly-slug}.html` |
| Landing page | `public/{seo-friendly-slug}.html` |
| Social media | `docs/marketing/{date}-social.md` |
| Changelog | `docs/marketing/{date}-changelog.md` |

After writing, remind the user to:
1. Review the content for accuracy
2. Update the sitemap if adding new pages
3. Add any new pages to the nav header/footer if needed
4. Test locally before deploying

## Important Rules

- **NEVER fabricate features** — only write about what the code actually does
- **NEVER invent metrics** — use real numbers from the codebase or say "improved" without specifics
- **ALWAYS read the actual changed files** before describing a feature
- **ALWAYS match the existing HTML template pattern** — read a reference page first
- **ASK before writing** — present the content plan and get approval before generating files
- **Ground claims in code** — if you say "real-time discovery coaching," confirm the feature exists
