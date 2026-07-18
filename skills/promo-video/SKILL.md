---
name: promo-video
description: Generate a structured promo video brief (script + shot list) for an RGA feature, grounded in actual code. Use when the user wants to create a 20s or 30s square promo video, typically after shipping a feature. The brief is written to Airtable and consumed by the rga-video-generator repo.
disable-model-invocation: true
argument-hint: "<feature-slug> \"<intent>\""
---

# RGA Promo Brief Generator

You produce a structured brief that a separate Remotion-based video generator
will turn into a 20- or 30-second promo. Your output is always grounded in
real code — never fabricated features.

The brief schema is defined in `skills/promo-video/brief-schema.md`. Read it
before drafting. Both the script and the shot list must validate against it.

## Inputs

- **feature-slug** (required): kebab-case slug, e.g. `meeting-prepper-v2-relaunch`.
- **intent** (required): 1-3 sentences explaining what to highlight. Drives
  creative angle and accent-word choice.

The user typically invokes this in conversational form, e.g.:

```
/promo-brief meeting-prepper-v2-relaunch "We just shipped Meeting Prepper V2.0
over the last couple of PRs. Reimagined research quality, prospect insights,
very actionable display. Game changer."
```

If either input is missing, ask for it directly. Do not guess.

## Airtable IDs (constants)

- Base: `app8WmO2rF14Bfv8O` (RGA Promo Pipeline)
- Features table: `tblRM5wMiuEtEsdpH`
- Promo Briefs table: `tblrFA4SLyRcbQ965`

Use the `airtable` MCP tools (`mcp__airtable__*`).

## Step 1: Resolve the Feature

Search the Features table by slug:

```
mcp__airtable__list_records(
  baseId="app8WmO2rF14Bfv8O",
  tableId="tblRM5wMiuEtEsdpH",
  filterByFormula="{Slug}='<feature-slug>'"
)
```

- **Found:** read its `Display Name`, `Description`, `Repo Path Hint`,
  `Product Area`, `Status`. Use these in subsequent steps.
- **Not found:** ask the user "I don't see this feature in Airtable yet.
  Want me to create it under which Product Area?" Offer the existing
  options: Meeting Prepper, Administration, Discovery, Proposal Creation.
  Also ask for the feature's current **codebase** Status — `planned`,
  `in-development`, or `shipped`. (This describes whether the code has
  merged to main, *not* the promo's state. Don't default; the right value
  depends on the feature.) On approval, create the record:

  ```
  mcp__airtable__create_record(
    baseId="app8WmO2rF14Bfv8O",
    tableId="tblRM5wMiuEtEsdpH",
    fields={
      "Slug": "<feature-slug>",
      "Display Name": "<derived-from-slug-or-asked>",
      "Product Area": "<chosen>",
      "Status": "<planned | in-development | shipped — ask user>"
    }
  )
  ```

  Do not proceed without a Features record.

## Step 2: Identify the Code

Use the feature's `Repo Path Hint` (if set) plus the user's intent to find
relevant files. Read them — components, routes, API handlers — to understand
the actual user experience.

If the user mentioned PRs ("over the last couple PRs", "in PR 123"):

```bash
gh pr list --state merged --limit 20 --search "<keyword-from-feature-or-intent>" \
  --json number,title,mergedAt,files
```

Present 3-5 candidate PRs with their titles and merge dates. Ask the user
to confirm which apply. Then read each confirmed PR's diff:

```bash
gh pr diff <number>
```

If the user explicitly listed PR numbers, skip the search and read those
directly.

## Step 3: Draft the Brief

Compose two JSON objects matching `skills/promo-video/brief-schema.md`.

### Script JSON (4 scenes)

- **problem**: pain point the user faces *without* this feature. Headline
  should sting. Lead with friction, not feature.
- **solution**: how the feature solves it. Often before/after screenshots.
  4 badges max showing concrete verified outcomes (e.g. "Verified email",
  "LinkedIn", "Title").
- **payoff**: the *why it matters* — outcome, not feature. Often emotional
  or outcome-focused.
- **cta**: brand line + button. Default button text is "DM us for a demo" —
  do NOT change unless the user explicitly asks.

Each headline must contain exactly one `{accent}...{/accent}` pair wrapping
the punch word(s). 1-4 words. See brief-schema.md for examples.

Pick `format`:
- `square-20` — default. Fits Instagram and LinkedIn natively.
- `square-30` — only if the feature genuinely needs more breathing room.

Pick `accent_color`:
- `green` — default. Matches RGA brand for "do this" / positive moments.
- `orange` — for "this is broken / pay attention" moments. Applied uniformly
  to all accents in the brief — not per-scene.

### Shot List JSON

One entry per unique screenshot referenced. Use kebab-case shot IDs that
match the scene references in the Script JSON.

For each shot, fill:
- `id`: kebab-case, unique within the brief.
- `description`: one sentence — what the shot shows.
- `route`: app path where this is captured, or `n/a` for non-app shots.
- `viewport`: default `"1080x1080 square selection"` — Matt screenshots a
  square region manually on Mac with Cmd+Shift+4.
- `state_setup`: imperative, actionable steps to reach the state ("Log in
  as demo user, navigate to /meeting-prepper, click first row").
- `what_to_capture`: what should be visible/highlighted in frame.
- `filename`: `{id}.png`.
- `drive_url`: null (filled in after upload).
- `uploaded`: false (flipped to true after upload).

## Step 4: Brand & Voice

- **Never mention third-party data providers** (e.g. PitchGhost, Clearbit,
  ZoomInfo). Focus on the outcomes the user sees.
- **Never invent features or metrics** — only describe what the code
  actually does.
- **Tone**: confident, direct, problem-first. "Stops X" / "Skips X" /
  "Finally" hooks. Avoid superlatives without specifics.
- The four scenes follow the structure: Problem → Solution → Payoff → CTA.
- Match the tone of the existing canonical example (Contact Data Enhancement,
  defined as `defaultProps` in `rga-video-generator/src/Root.tsx`).

## Step 5: Present Inline & Iterate

Show the full draft in chat as TWO blocks:

1. **Plain-English readback** of each scene's headline + subtext, so the user
   can read it without parsing JSON. Format:

   ```
   ## Draft Brief: <Feature Display Name>

   **Format:** square-20  |  **Accent color:** green

   **Scene 1 — Problem (5s)**
   Headline: Most sales reps **skip** prospect research.
   Subtext:  Not because they don't care — because they can't find the basics.
   Screenshot: problem-empty-form

   **Scene 2 — Solution (6s)**
   Headline: RGA finds it for you. **One click.**
   Before:   solution-button
   After:    solution-enriched
   Badges:   Verified email · LinkedIn · Title · Company details

   ... (continue for all 4 scenes)
   ```

   Bold the accent words. Show scene durations from the format recipe.

2. **Raw JSON** — both Script JSON and Shot List JSON in fenced code blocks
   so the user can spot-check against the schema.

3. **Shot list summary** — quick count of unique shots and what they need
   to capture, formatted as a numbered checklist.

Do NOT write to Airtable yet.

Ask: "How does this read? Anything to adjust?"

Iterate. Common adjustments:
- "Make the problem sting more" → tighter, more visceral headline
- "Lose the badges, too much" → drop or shorten badges
- "Wrong accent word" → re-mark headline
- "30 seconds" → re-pace, switch format to `square-30`
- "Add a shot of X" → extend shot list, reference from a scene

When the user approves the script + shot list, say "Script locked. Now
drafting LinkedIn and Instagram captions." Move to Step 6.

## Step 6: Draft Captions

Now (and only now, after the script is locked) draft both captions per
the schema in `brief-schema.md` (see "Captions" section). Read it before
drafting; it has hard rules on length, structure, and tone.

### Tone rules — NON-NEGOTIABLE

These are dealbreakers. Every caption must pass these checks:

- **No em-dashes (—).** Use a comma, period, colon, or rewrite. This is
  the single most reliable AI-writing tell. If you write one, delete it.
- **No filler verbs:** leverage, unlock, elevate, empower, revolutionize,
  seamlessly, robust, game-changer, synergy.
- **No "in today's fast-paced world"** or any variant. No "we're excited
  to announce." Just say what shipped.
- **No rhetorical questions** as the hook ("Tired of X?"). Make a claim.
- **No decorative three-item lists** ("faster, smarter, better"). Lists
  are fine when concrete and real.
- **Active voice.** No "is equipped with," "enables you to," "allows
  for." Subject does the verb.
- **No invented metrics.** If "90 seconds" isn't measured, don't say it.
- **Match the script.** If the script is blunt, the captions are blunt.
  Do not pivot to LinkedIn-thinkfluencer voice.

If a sentence sounds like every other AI-tool LinkedIn post, delete it.
This is the test: would a human who doesn't post for a living write
this exact sentence? If no, rewrite.

### Compose

Build `captions.linkedin` and `captions.instagram` per the schema:

- **LinkedIn:** `hook` (≤150 char, what's visible before "see more"),
  `body` (2-4 short paragraphs, total post 600-1300 char range),
  `hashtags` (**2-3 max**, PascalCase, no `#` prefix), `cta` (one line,
  default "DM me for a demo.").
- **Instagram:** `caption` (≤200 char before hashtags, first line
  hooks), `hashtags` (**5-7**, lowercase, no separator dots before
  the trailing block).

### Present

Show the user BOTH captions in their final concatenated form (the way
they'd appear when posted), not the JSON. Format:

```
## Drafted Captions

### LinkedIn (will save to `LinkedIn Caption` field)

<hook>

<body — preserve paragraph breaks>

<cta>

#<tag> #<tag> #<tag>

---

### Instagram (will save to `Instagram Caption` field)

<caption>

#<tag> #<tag> #<tag> #<tag> #<tag>
```

Then ask: "How do these read? Anything to adjust?"

Iterate. Common adjustments:
- "Hook is weak" → rewrite the LinkedIn hook
- "Too long" → cut to essentials
- "Sounds AI-generated" → audit against tone rules above, rewrite
- "Different angle" → re-anchor on a different beat from the script

## Step 7: Confirm Before Writing

Once both script AND captions are approved, ask explicitly:

> "Ready to write this brief to Airtable? (y/n)
>  This creates a Promo Briefs record with status `draft`, including
>  the script, shot list, and both captions."

If yes, proceed to Step 8. If no, stay in chat for further edits.

## Step 8: Write to Airtable

Build the final Script JSON with `captions` embedded:

```json
{
  "version": 1,
  "format": "...",
  "accent_color": "...",
  "scenes": { ... },
  "captions": {
    "linkedin": { "hook": "...", "body": "...", "hashtags": [...], "cta": "..." },
    "instagram": { "caption": "...", "hashtags": [...] }
  }
}
```

Build the formatted caption strings (these go in the dedicated Airtable
fields for one-click copy-paste). No `#` prefix in the JSON arrays; add
it in concatenation:

```
linkedin_formatted =
  hook + "\n\n" + body + "\n\n" + cta + "\n\n" + "#" + hashtags.join(" #")

instagram_formatted =
  caption + "\n\n" + "#" + hashtags.join(" #")
```

Then create the Airtable record:

```
mcp__airtable__create_record(
  baseId="app8WmO2rF14Bfv8O",
  tableId="tblrFA4SLyRcbQ965",
  fields={
    "Name": "<Feature Display Name> — <short angle>",
    "Feature": ["<features-record-id>"],
    "Status": "draft",
    "Intent": "<user's intent statement>",
    "Script JSON": "<stringified script JSON, with captions embedded>",
    "Shot List JSON": "<stringified shot list JSON>",
    "LinkedIn Caption": "<linkedin_formatted>",
    "Instagram Caption": "<instagram_formatted>",
    "PR Reference": "<one PR URL per line>",
    "Notes": ""
  }
)
```

**Naming convention:** `<Feature Display Name> — <angle>`. Examples:
- `Meeting Prepper — V2.0 relaunch, research quality angle`
- `Contact Data — verified-email hero`

After creating, confirm the new record's URL to the user.

## Step 9: Create ClickUp Task

Always create a ClickUp task in the RGA Campaign list to track the promo
through to production. Assign it to Matt Oess and set a due date 3 days
from today.

```
mcp__clickup__createTask(
  list_id="901409083855",
  name="<same as Airtable record name>",
  status="planned",
  tags=["promo-video", "<product-area-lowercased-with-dashes>"],
  assignees=["Matt Oess"],
  due_date="<today + 3 days, ISO 8601 — e.g. 2026-05-06>",
  description="Brief: <Airtable record URL>\n\nIntent: <intent statement>\n\nNext: capture screenshots per shot list, then run /build-promo in rga-video-generator."
)
```

Then write the resulting ClickUp task URL back to the Airtable record's
`ClickUp Campaign URL` field.

## Step 10: Hand-off Instructions

Wrap up with this exact block (substitute `<...>` placeholders):

```
✓ Brief saved to Airtable: <record URL>

NEXT STEPS:

1. Capture screenshots:
   - Use Cmd+Shift+4 on Mac for a precise square selection.
   - For each shot in the shot list, follow its `state_setup` and capture
     what `what_to_capture` describes.
   - Save with the exact `filename` from the shot list.

2. Upload to Google Drive:
   - Folder path: RGA Marketing → RGA Campaigns → <campaign-folder> → screenshots → <feature-slug>/
   - Right-click each file → Share → "Anyone with the link" → copy link.

3. Update the brief in Airtable:
   - For each shot, paste the Drive share URL into `drive_url` and set
     `uploaded: true` in the Shot List JSON.
   - Change the brief's Status from `draft` to `screenshots-ready`.

4. Build the video:
   - cd ~/Documents/Claude/rga-video-generator
   - Run: /build-promo "<brief name or Airtable record id>"
```

## Important Rules

- NEVER write to Airtable without explicit user approval.
- NEVER fabricate features, metrics, or capabilities.
- ALWAYS read actual code (components, routes, handlers) before drafting.
- ALWAYS validate Script + Shot List JSON against `brief-schema.md` before
  writing. Every accent marker present, every shot reference resolves.
- ALWAYS present drafts inline (plain English + JSON) before the Airtable
  write.
- The default button text is `"DM us for a demo"`. Do not change it.
- Never mention third-party data providers in the script.
- If the feature slug doesn't exist in Airtable, create the Features record
  with user approval before proceeding. Don't write a brief without one.

### Caption-specific dealbreakers

- **No em-dashes anywhere in captions.** Audit before presenting. If your
  draft contains one, fix it before the user sees it.
- **No corporate filler verbs.** leverage, unlock, elevate, empower,
  revolutionize, seamlessly, robust, game-changer, synergy.
- **No "we're excited / thrilled / proud to announce."** Just state what
  shipped.
- **No rhetorical questions as the LinkedIn hook.** Make a claim.
- **Active voice, no passive constructions.**
- **Match the script's tone.** Don't pivot to LinkedIn-thinkfluencer voice.
- **No markdown emphasis** (`*italics*` / `**bold**`) in LinkedIn
  captions. LinkedIn does not render markdown; the asterisks display
  literally and read as an AI/copy-paste tell.
- **Hashtag counts are HARD limits.** LinkedIn: 2-3. Instagram: 5-7.
  More hashtags do not help in 2026.
- **No `.\n.\n.\n` separator dots** before Instagram hashtag blocks.
  Single blank line only.

If a caption sentence sounds like every other AI-tool LinkedIn post,
rewrite it. The whole point of these captions is they don't read as
AI-generated.
