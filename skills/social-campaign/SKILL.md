---
name: social-campaign
description: Generate a complete on-brand social campaign (LinkedIn + Instagram carousel + copy) for an RGA product update, blog post, or topic, grounded in the Marketing Brain, then build the carousel slides as HTML mocks rendered to PNG. Use when the user wants to turn something shipped into ready-to-publish social assets.
disable-model-invocation: true
argument-hint: "<PR # / blog URL / topic>  (optional; will prompt if omitted)"
---

# RGA Social Campaign Generator

You turn a product update, blog post, or topic brief into a stunning, on-brand,
viral-friendly campaign for LinkedIn AND Instagram — copy plus built carousels —
in a single session on this machine. Your output is always grounded in real,
verified facts: never fabricate a feature, number, or customer.

This skill operationalizes two companion docs. **Read both before drafting:**
- `docs/marketing/RGA-Social-Playbook.md` — the WHY (hook frameworks, slide
  templates, copy rules, visual system, red flags). This is your rulebook.
- `docs/marketing/RGA-Campaign-Workflow.md` — the HOW (Canva IDs, tool
  reference, step-by-step). This is your operational companion.

And the source of truth for every claim:
- `docs/marketing/RGA-Marketing-Brain.md` — every number, feature, and quote you
  put on a slide MUST trace to a citation here. If a claim has no mapping in the
  Brain, either drop it or get it verified first. No exceptions. This is the
  single rule that separates this workflow from AI slop.

## Ground rules (do not violate)

1. **Brain-grounded or it doesn't ship.** Map every factual claim to Brain
   Section 1 (Product Truth), 2 (Public Claims), or 3 (Verified Math). Run the
   whole draft against Brain Section 5 (Do-Not-Say) and strip anything forbidden.
2. **No em-dashes in reader-facing copy** (LLM tell). Use periods or parentheses.
3. **No engagement bait, no AI-slop phrases, no emoji bullets.** See Playbook
   Part 9. LinkedIn's NLP classifier suppresses bait ~60%.
4. **Both platforms, always.** Every concept ships as a LinkedIn translation AND
   an Instagram translation (Playbook Part 6).
5. **Build slides as HTML, not in Canva.** Render to PNG at 2400x2400. Canva is
   a fallback for hand-editable decks (Step 6b), not the default.
6. **Show, do not tell.** Every carousel needs real product output at full
   fidelity, anonymized. Pull genuine examples from Airtable rather than
   inventing them, and never present invented output as real.

## The flow

### Step 1: Intake
If arguments were provided (a PR #, blog URL, or topic), use them. Otherwise ask
(via AskUserQuestion) for the missing pieces:
- **Source:** PR link, product-update note, blog URL, or raw idea?
- **Goal:** demo signups, brand awareness, thought leadership, or save/share velocity?
- **Audience:** RevOps leaders, VP Sales, founders, or SDR managers?
- **Anchor:** any specific customer story, number, or quote to build around?
- **Publish window:** this week, next sprint, or evergreen?

### Step 2: Gather ground truth
Pull the real material so claims are accurate:
- If a PR #: `gh pr view <n> --json title,body,files` and read the actual diff
  for what shipped. Cross-check against the Brain.
- If a blog URL: fetch it; anchor to its most contrarian line.
- If a topic: map it to Brain Section 1 features and Section 3 math.
- Note which of the four **unique-access** sources (Playbook "The Moat") this
  post draws from — agent execution data, PR history, founder POV, or customer
  call data. If it draws from none, say so and reconsider the angle.

### Step 3: Concept + hook
Pick ONE hook framework from Playbook Part 2 based on post type:
- Product update → Problem-Agitate-Solve or Before/After
- Blog promotion → Numbered List or Contrarian Take
- Customer win → Before/After or Curiosity Gap
- Strategic / manifesto → Founder POV or Contrarian
- Process / story → Behind-the-Scenes

Draft **3 hook options** in the chosen framework and present them for selection
before writing the rest. (For Numbered List, each option must pass all four
conditions in Playbook Part 2E or don't offer it.)

### Step 4: Copy
Once a hook is approved, write:
- **LinkedIn body** — 1,300 to 1,900 chars, formatted per Playbook Part 4
  (hook under 120 chars above the fold, whitespace, arrow/dot bullets, soft
  open question to close). Link goes in the FIRST COMMENT, never the body.
- **Instagram caption** — 125-char front-loaded punch; expand to 800 to 1,500
  if educational. "Link in bio" only.
- **First-comment text** for the LinkedIn link.
- **Hashtags** — 3 for LinkedIn (broad/mid/niche mix), 3 to 5 for Instagram.

### Step 5: Carousel script (the slide spec)
Generate slide-by-slide content using the templates in Playbook Part 3. This
spec is the artifact that drives the slide build, so make it precise. For each
slide, emit exactly:

```
SLIDE <n>
Role:          <slide role from the Part 3 template>
Headline:      <text, within the word budget for that slide>
Body:          <text if any, max 30 words>
Design note:   <visual treatment — which UI screenshot / illustration / pure type>
Brand element: <logo? slide number "n of 7"? color block?>
```

- LinkedIn: 7-slide structure at 1080x1350 (product update / blog / educational
  template as appropriate).
- Instagram: 8-slide structure, mirroring LinkedIn, with a note on which slide
  should be motion/video (typically slide 3 — IG tracks reach-to-slide-3).
- If a slide references a product screenshot, note exactly which UI element and
  crop. Screenshots must follow Playbook Part 5 DO/DON'T (crop tight, annotate
  in brand color not red, anonymize customer data).

### Step 6: Build the slides as HTML mocks, render to PNG (DEFAULT)

**This is the default path. It was validated end to end on 2026-07-18 and it
beats driving Canva.** Existing Canva bases are built for short feature blurbs;
mapping a teaching script onto them forces word-count compression that strips
the fidelity that earns engagement, and their baked-in icons/screenshot frames
cannot be fixed through the MCP (no image-generation, no add-page).

Work in `docs/marketing/carousel-mocks/` in the ai-sales-agent-v5 repo:
- `_base.css` — shared tokens: dark navy gradient
  (`radial-gradient(ellipse at 20% 0%, #1e3a6f, #12245a, #0a1642)`), RGA orange
  `#fe6601`, accent blue `#7fb0ff`, Inter, `.pill` / `h1` / `.sub` / `.foot`.
- `slideN-<name>.html` — one file per slide, `@import url('_base.css')` then a
  small per-slide `<style>` block, then the copy as plain text.
- `render.mjs` — Playwright renderer. Run from the repo root so `playwright`
  resolves. Viewport 1200x1200, `deviceScaleFactor: 2`, clipped to
  1200x1200 → 2400x2400 PNG.

```
node docs/marketing/carousel-mocks/render.mjs \
  docs/marketing/carousel-mocks/slide2-opener.html \
  docs/marketing/carousel-mocks/slide2-opener.png
```

Design rules that made this work:
- **The screenshot is the hero.** Put a real product window (browser chrome,
  the actual prep content) at full fidelity and let text serve it. Dense,
  pinch-zoomable slides outperform big-word slides.
- **Never compress the content to fit the frame.** Resize the frame.
- Match the real product styling: white card, `#1B365D` headings,
  `#fe6601` accents (see `src/components/discovery/MeetingPrepperReport.tsx`).
- Secondary text needs to be large: `.sub` at ~32px minimum, notes 25px+.
  Anything sized for desktop reading is unreadable on a phone.
- **Always run an overflow check** after type changes: load each page in
  Playwright and assert no element's `bottom`/`right` exceeds 1201px.

Then publish a **review page** (Artifact) with every slide plus its notes, so
the user can pinch-zoom on a phone. Inline chat images are NOT a reliable
review channel. Revise the HTML and re-render until approved.

### Step 6b (optional): Canva
Keep Canva for decks the user wants to hand-edit or hand off, or for one-off
graphics. Copy an existing base and edit it:
`copy-design` → `start-editing-transaction` → `perform-editing-operations` →
`commit-editing-transaction`.
- Bases: `DAHEVQ0KtU0` 2026 Spring Launch (7pp, dark navy, image-layer based),
  `DAGuj7n4DkA` 6 ways RGA (7pp, 1080x1350, text-editable),
  `DAGjUZGn07M` RGA Square, `DAGulE6p8zg` Before/After.
- RGA Brand Kit `kAGndsNYYbg`. Use `resize-design` for wrong dimensions.
- **Inspect a base's real text capacity before mapping a script onto it.**
- Known limits: no add-page, no image generation, and
  `upload-asset-from-url` requires an already-public URL, so locally rendered
  PNGs cannot be pushed into Canva without publishing them first. Do not do
  that with embargoed material; have the user drag the PNGs in instead.

### Step 7: Pre-publish QA
Run the full checklist from Playbook Part 8 against BOTH variants. Do not skip:
slide 1 swipe-worthiness, hook length, one-idea-per-slide, no bait/slop/emoji
bullets, brand blocks, link-in-first-comment, hashtag limits, a real
number/customer/POV present, CTA matches post type, both variants exist.

### Step 8: Handoff to ClickUp
Create a task in the RGA Campaign list (`901409083855`, Space "Ai Sales Agent",
folder Marketing and Sales):
- Title: the campaign name
- Status/anchor line: "Ready to publish"
- Body: both final copies, first-comment text, hashtags, Canva view+edit URLs
  for LinkedIn and IG, recommended publish day/time (Playbook Part 7), and the
  Brain citations backing each factual claim.

### Step 9: Publish to GoHighLevel
Verify connected accounts (`social-media-posting_get-account`), then
`social-media-posting_create-post` per piece. Confirmed account IDs:
- LinkedIn Matt Oess: `67ddd7fe838e587a959cab49_zTPFln9s8sc8hUWqgr4q_aW9gpZq5aK_profile`
- LinkedIn RGA page: `67ddd7fe838e587a959cab49_zTPFln9s8sc8hUWqgr4q_107978108_page`
- Instagram mattoess: `67ddd7ba031a9629acb54252_zTPFln9s8sc8hUWqgr4q_17841402062630118`
- (Justeen's LinkedIn token is expired; do not target it.)

Put the LinkedIn link in `followUpComment`, never the body. Create as
`status: "draft"` unless the user explicitly asks to schedule.

**Media limitation:** GHL has no direct file-upload path here, so locally
rendered PNGs cannot be attached programmatically. Create the draft with the
copy, then tell the user to drag the PNGs from
`docs/marketing/carousel-mocks/` into the Social Planner. Do NOT publish the
images to a public host to work around this.

## Standing inputs (reuse)
- RGA Brand Kit: `kAGndsNYYbg` · TechCXO Brand Kit: `kAFq-74Ighk`
- ClickUp: list `901409083855` (RGA Campaign), space "Ai Sales Agent"
- Optimal windows: LinkedIn Tue/Wed 10am-4pm ET; Instagram Tue-Thu 11am-1pm / 7-9pm

## What NOT to do
- Do not add any executable code to `api/` or touch `vercel.json`. This is a
  Claude skill, not a service. It stays entirely in `.claude/` and `docs/`.
- Do not ship a claim that isn't in the Brain.
- Do not post to GHL unless the user explicitly asks (Step 9 is opt-in).
