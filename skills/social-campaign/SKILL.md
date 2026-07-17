---
name: social-campaign
description: Generate a complete on-brand social campaign (LinkedIn + Instagram carousel + copy) for an RGA product update, blog post, or topic, grounded in the Marketing Brain, then build the carousels in Canva via MCP. Use when the user wants to turn something shipped into ready-to-publish social assets.
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
5. **Reuse existing designs.** Build carousels by copying a proven RGA base
   design and editing its slides. Do not generate net-new unless the user asks.

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
spec is the artifact that drives the Canva build, so make it precise. For each
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

### Step 6: Build in Canva (via MCP, this machine)
Confirm the Canva connector responds (`search-designs` query "RGA"). Then, per
`RGA-Campaign-Workflow.md`:

**Preferred — copy an existing strong base and edit it:**
- Reference designs (verified reachable):
  - `DAHEVQ0KtU0` — 2026 Spring Launch RGA (7 pages, IG carousel)
  - `DAGuj7n4DkA` — 6 ways RGA (7 pages, 1080x1350)
  - `DAGjUZGn07M` — RGA Square 1080x1080 (1 page)
  - `DAGulE6p8zg` — Before After (1 page)
  - `DAHIQnSvhkk` / `DAHIQlKotGo` — Research-hurdle presentations (4 pages)
- Pick the base whose structure best matches the script, then:
  `copy-design` → `start-editing-transaction` → `perform-editing-operations`
  (swap each slide's text/visuals from the Step 5 spec) → `commit-editing-transaction`.
- Upload any referenced screenshots as assets and place them per the design notes.
- RGA Brand Kit: `kAGndsNYYbg` (for color/type when editing or generating).

**Only if no base fits:** `generate-design` from brand kit `kAGndsNYYbg`.

Produce **both** platform variants (Playbook Part 6). Use `resize-design` if a
base is the wrong dimensions.

Note: some designs are export-locked (`disableexport=T`). Copying and editing
still work; only `export-design` may be restricted. If export is blocked, hand
back the edit/view URLs and note that the user exports manually.

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

### Step 9 (optional): Schedule into GoHighLevel
Only if the user asks. Verify connected accounts
(`social-media-posting_get-account`), then `social-media-posting_create-post`
for each piece at the optimal window, confirm via `get-posts`, and update the
ClickUp task to "Scheduled" with the GHL post IDs.

## Standing inputs (reuse)
- RGA Brand Kit: `kAGndsNYYbg` · TechCXO Brand Kit: `kAFq-74Ighk`
- ClickUp: list `901409083855` (RGA Campaign), space "Ai Sales Agent"
- Optimal windows: LinkedIn Tue/Wed 10am-4pm ET; Instagram Tue-Thu 11am-1pm / 7-9pm

## What NOT to do
- Do not add any executable code to `api/` or touch `vercel.json`. This is a
  Claude skill, not a service. It stays entirely in `.claude/` and `docs/`.
- Do not ship a claim that isn't in the Brain.
- Do not post to GHL unless the user explicitly asks (Step 9 is opt-in).
