# Promo Video Build Procedure

This is the build-side of the promo-video workflow. The brief was created
by the `/promo-brief` command and lives in Airtable. Your job is to turn
an approved brief (status `screenshots-ready`) into a registered Remotion
composition that Matt can preview and render.

The brief schema is at `skills/promo-video/brief-schema.md`. The video
template (`FeaturePromo`) is at `src/compositions/FeaturePromo.tsx` in the
`rga-video-generator` repo. The TypeScript schema + Zod validators are at
`src/lib/promoSchema.ts`.

## Working directory

You must be running in `/Users/MattOess/Documents/Claude/rga-video-generator`.
If `pwd` is not that path, halt and tell the user to `cd` there first.

## Airtable IDs (constants)

- Base: `app8WmO2rF14Bfv8O`
- Promo Briefs table: `tblrFA4SLyRcbQ965`
- Features table: `tblRM5wMiuEtEsdpH`

## Step 1: Resolve the Brief

If the user passed a record id (matches `^rec[A-Za-z0-9]{14}$`), fetch directly:

```
mcp__airtable__get_record(baseId="app8WmO2rF14Bfv8O", tableId="tblrFA4SLyRcbQ965", recordId=...)
```

Otherwise treat the argument as a name and search:

```
mcp__airtable__list_records(
  baseId="app8WmO2rF14Bfv8O",
  tableId="tblrFA4SLyRcbQ965",
  filterByFormula="FIND(LOWER('<query>'), LOWER({Name}))"
)
```

If multiple matches, list them and ask the user to pick. If none, list
recent briefs with status `screenshots-ready` for the user to choose.

## Step 2: Validate Status

The brief's `Status` must be `screenshots-ready`. Anything else means
either the work isn't done (`draft`, `ready-for-screenshots`) or it's
already past this stage (`in-production`, `rendered`, `published`,
`archived`).

If status ≠ `screenshots-ready`:
- Tell the user the current status.
- If `draft` or `ready-for-screenshots`: tell them to capture & upload
  screenshots first, then update the brief and shot list.
- If `in-production` or later: ask whether they want to rebuild (force
  flag), or pick a different brief.
- Halt unless explicitly told to override.

## Step 3: Parse & Validate JSON

Read `Script JSON` and `Shot List JSON` from the brief record. Both are
stored as stringified JSON in long-text fields. Parse with `JSON.parse`.

Validate structurally against `brief-schema.md`:

**Script JSON:**
- `version === 1`
- `format` is one of the supported values (currently `square-20`,
  `square-30` — defined in `src/lib/promoSchema.ts` `FORMAT_RECIPES`).
- `accent_color` is `green` or `orange`.
- `scenes.problem`, `scenes.solution`, `scenes.payoff`, `scenes.cta` all
  present.
- Every `headline` contains exactly one `{accent}...{/accent}` pair.
- `scenes.solution.badges` is an array of ≤ 4 strings.
- `scenes.cta.button_text` is ≤ 22 characters.

**Shot List JSON:**
- `version === 1`
- `shots` is a non-empty array.
- Every shot has all required fields.
- Every `id` is kebab-case and unique.
- Every `filename` matches `^[a-z0-9-]+\.png$`.

**Cross-validation:**
- Every shot referenced by a scene (`screenshot`, `before_screenshot`,
  `after_screenshot`) exists in the shot list.
- Every shot in the list is referenced by at least one scene (warning,
  not fatal).

If validation fails, list every problem and halt. Do not proceed.

## Step 4: Verify Uploads

For every shot in the list:
- `uploaded` must be `true`.
- `drive_url` must be a non-null URL.

If any fail, list the missing ones and halt:

```
The following shots are not yet uploaded:
  - <shot-id>: <description>
  - <shot-id>: <description>

Update the Shot List JSON with `drive_url` and `uploaded: true` for each,
then re-run /build-promo.
```

## Step 5: Download Screenshots to public/

For each shot, download from `drive_url` to `public/{filename}`.

Google Drive share URLs come in two forms:
- `https://drive.google.com/file/d/{ID}/view?usp=sharing` (most common)
- `https://drive.google.com/open?id={ID}`

Convert to a direct-download URL: `https://drive.google.com/uc?export=download&id={ID}&confirm=t`.

```bash
curl -L "https://drive.google.com/uc?export=download&id=<ID>&confirm=t" \
  -o public/<filename>
```

After each download, verify the file exists and is non-trivially sized
(> 1 KB). If a download is HTML (Google's "can't scan for viruses"
intercept), use the `&confirm=t` form. If it still fails, ask the user
to manually download and place the files in `public/`.

After all downloads, list the files saved.

## Step 6: Generate Composition ID

The Composition ID is the React/Remotion identifier and the render target.
PascalCase, derived from the brief's `Name` field (or feature slug +
angle).

Examples:
- "Meeting Prepper — V2.0 relaunch, research quality angle" →
  `MeetingPrepperV2RelaunchResearchQuality` (or shorter: `MeetingPrepperV2`)
- "Contact Data — verified-email hero" → `ContactDataVerifiedEmail`

Default to a reasonable shortening, then ask the user: "Composition ID
will be `<X>`. Press Enter or suggest different."

The ID must be unique across `src/Root.tsx` (search for existing
`<Composition id="..."` entries first).

## Step 7: Add Composition Entry to Root.tsx

Read `src/Root.tsx`. Find the closing `</>` of the `RemotionRoot` fragment
and add a new entry just before it:

```tsx
{/*
  ── <Display Name> ──────────────────────────────────────────────
  Brief: <Airtable record URL>
  Format: <format>
  Render: npx remotion render src/index.ts <CompositionID> out/<id-kebab>.mp4
*/}
<Composition
  id="<CompositionID>"
  component={FeaturePromo}
  durationInFrames={totalFrames("<format>")}
  fps={FORMAT_RECIPES["<format>"].fps}
  width={FORMAT_RECIPES["<format>"].width}
  height={FORMAT_RECIPES["<format>"].height}
  defaultProps={{ brief: <variableName> }}
/>
```

Add the brief object as a `const <variableName>: Brief = { ... }` near
the top of the file, alongside `contactDataBrief`. Use the parsed (not
stringified) script + shotList objects.

Imports already include `FeaturePromo`, `totalFrames`, `FORMAT_RECIPES`,
and `Brief` — do not duplicate.

## Step 8: Type Check

```bash
npx tsc --noEmit
```

If errors, fix them and re-check. Common issues:
- Missing comma in JSON-derived object literal
- Brief variable name collision
- Unused import

Do not proceed until clean.

## Step 9: Update Airtable

```
mcp__airtable__update_records(
  baseId="app8WmO2rF14Bfv8O",
  tableId="tblrFA4SLyRcbQ965",
  records=[{
    "id": "<brief record id>",
    "fields": {
      "Status": "in-production",
      "Composition ID": "<CompositionID>"
    }
  }]
)
```

## Step 10: Hand-off

Output exactly this block, with substitutions:

```
✓ Promo composition wired up.

Composition ID: <CompositionID>
Format: <format> (<W>×<H>, <duration>s)
Brief: <Airtable record URL>

NEXT STEPS:

1. Preview in Remotion Studio:
     npm run dev
   Open the composition in the sidebar. Scrub through, watch all 4 scenes.

2. Iterate as needed:
   - Script edits: update the brief in Airtable, then re-run /build-promo
     (it will overwrite the Composition entry). OR edit the brief object
     directly in src/Root.tsx for quick local tweaks (sync back to Airtable
     after).
   - Screenshot fixes: re-capture, re-upload to Drive, update the Shot
     List JSON, re-run /build-promo.

3. Render the final video:
     npx remotion render src/index.ts <CompositionID> out/<id-kebab>.mp4

4. After render:
   - Upload the MP4 to Google Drive: RGA Marketing → Campaigns → <campaign>/videos/
   - Paste the share URL into the Airtable brief's `Output MP4 URL` field.
   - Update Status to `rendered`.
   - When published, update Status to `published`.
```

## Important Rules

- NEVER proceed past Step 4 if any shot is missing.
- NEVER edit `src/lib/promoSchema.ts` or `src/compositions/FeaturePromo.tsx`
  as part of /build-promo. Those are template code; if they need changes,
  it's a separate task.
- ALWAYS run typecheck before reporting success.
- ALWAYS update Airtable status after a successful wire-up — that's the
  signal to the rest of the workflow.
- Composition IDs must be unique. If a brief is being rebuilt, replace
  the existing Composition entry rather than adding a duplicate.
- If the user is in the wrong directory (not `rga-video-generator`),
  halt early — don't try to be clever about cd-ing.
