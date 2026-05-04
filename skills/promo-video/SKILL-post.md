# Promo Video Post Procedure

This is the publish-side of the promo-video workflow. The brief was
created by `/promo-brief`, the video was built and rendered by
`/build-promo`, and now it's ready to ship to GoHighLevel.

You do **not** post directly to LinkedIn or Instagram. You send the post
to GoHighLevel's Social Planner as drafts (or pre-scheduled drafts if a
publish date is set). The user then polishes inside GHL — line breaks,
person tags, polls, video preview — and clicks publish.

The actual GHL API call is handled by a Make.com scenario. This skill
validates the brief, formats the payload, and fires the webhook.

## Working directory

You can run this from either repo:
- `~/Documents/ai-sales-agent-v5/` (where briefs are authored), or
- `~/Documents/Claude/rga-video-generator/` (where videos are built).

The skill is the same. Don't worry about which repo.

## Airtable + Make constants

- Base: `app8WmO2rF14Bfv8O` (RGA Promo Pipeline)
- Promo Briefs table: `tblrFA4SLyRcbQ965`
- Make webhook: read from `MAKE_GHL_PROMO_WEBHOOK` env var. If unset,
  halt and tell the user to add it to `.env`.

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

If no argument is provided, list recent briefs with status `rendered`
and ask the user to pick one.

## Step 2: Validate State

The brief must be ready to post. Check:

- **Status** is `rendered`. If anything else (especially `posted-as-drafts`
  or `live`), warn the user and ask whether to re-send (which will create
  duplicate drafts in GHL).
- **`Output MP4 URL`** is set and non-empty. (The video is rendered and
  uploaded to Drive.)
- **`LinkedIn Caption`** is set and non-empty.
- **`Instagram Caption`** is set and non-empty.
- **At least one platform in `Posted To`** OR no platforms set yet
  (interpret empty as "post to all platforms with captions").

If any of these fail, list every problem and halt.

## Step 3: Decide Platforms

The brief's `Posted To` field is a multi-select. Treat it as the set of
platforms TO post to:

- **If empty:** post to all platforms with captions present (typically
  both LinkedIn and Instagram). Tell the user "Posting to LinkedIn and
  Instagram (default since `Posted To` is empty)."
- **If set:** only post to those listed. If `Posted To` includes a
  platform with no caption present, halt with an error.

## Step 4: Decide Mode (draft vs scheduled draft)

Check the `Publish Date` field:

- **If empty:** mode is `"draft"`. GHL receives the post as a plain draft.
  The user opens GHL and schedules/publishes manually.
- **If set:** mode is `"scheduled-draft"`. GHL receives the post as a
  draft AND with a scheduled publish time. The user can polish before
  the time hits; if untouched, GHL publishes automatically at the time.

For scheduled mode, normalize the timestamp to ISO-8601 with timezone:
`2026-05-05T14:00:00-04:00` (Airtable returns UTC; convert to whatever
timezone the user's account is configured for, default America/New_York).

## Step 5: Lint Captions Before Sending

Audit both captions one more time before showing the user:

- **LinkedIn caption:**
  - No em-dashes (`—`).
  - No markdown emphasis (`*foo*` or `**foo**`). LinkedIn renders these
    literally.
  - Hashtag count: 2-3 trailing tags.
- **Instagram caption:**
  - No em-dashes.
  - No `.\n.\n.\n` separator dots before the hashtag block.
  - Hashtag count: 5-7 trailing tags.

If anything fails, list the specific problems and ask: "These captions
have issues. Fix in Airtable then re-run, or send anyway?"

Do NOT silently auto-correct. The user owns the copy.

## Step 6: Show Preview

Show the user EXACTLY what will be sent, per platform. No JSON. Format:

```
## Sending to GHL Social Planner

**Mode:** Draft  (or: Scheduled draft for May 5, 2026 at 2:00 PM ET)
**Brief:** Meeting Prepper V2.0 — relaunch, whole-picture angle
**Video:** [Drive link]
**Location:** RGA TechCXO (configured in Make scenario)

---

### LinkedIn

[exact caption text as it will appear in GHL]

---

### Instagram

[exact caption text as it will appear in GHL]
```

Then ask: "Send these to GHL? (y/n)"

If no, stay in chat for further edits to Airtable before re-running.

## Step 7: Fire the Make Webhook

On approval, POST to the Make webhook with this exact payload shape:

```json
{
  "brief_id": "recerNJCKtxgjDGbY",
  "brief_name": "Meeting Prepper V2.0 — relaunch, whole-picture angle",
  "mode": "draft",
  "scheduled_at": null,
  "video_drive_url": "https://drive.google.com/file/d/.../view?usp=drivesdk",
  "video_drive_id": "1mmeyB3QncOfD-TryWT-pWAGew1f5HnE1",
  "platforms": ["LinkedIn", "Instagram"],
  "captions": {
    "LinkedIn": "<exact LinkedIn Caption text>",
    "Instagram": "<exact Instagram Caption text>"
  },
  "airtable_record_url": "https://airtable.com/app8WmO2rF14Bfv8O/tblrFA4SLyRcbQ965/recerNJCKtxgjDGbY"
}
```

Notes on the payload:
- `mode`: `"draft"` or `"scheduled-draft"`.
- `scheduled_at`: ISO-8601 timestamp with timezone if mode is
  `scheduled-draft`, otherwise `null`.
- `video_drive_id`: extracted from the Drive URL (between `/d/` and
  `/view`). Make uses this to fetch the file from Drive directly.
- `platforms`: the validated list from Step 3.
- `captions`: keyed by platform name; only includes platforms in
  `platforms`.

Use:

```bash
curl -X POST -H "Content-Type: application/json" \
  -d '<payload>' \
  "$MAKE_GHL_PROMO_WEBHOOK"
```

The Make scenario should respond synchronously with a JSON body
containing per-platform draft URLs:

```json
{
  "ok": true,
  "drafts": {
    "LinkedIn": "https://app.gohighlevel.com/.../social-planner/posts/<id>",
    "Instagram": "https://app.gohighlevel.com/.../social-planner/posts/<id>"
  }
}
```

If the response is an error or the webhook times out (>30s), halt and
tell the user. Don't update Airtable.

## Step 8: Update Airtable

On webhook success:

```
mcp__airtable__update_records(
  baseId="app8WmO2rF14Bfv8O",
  tableId="tblrFA4SLyRcbQ965",
  records=[{
    "id": "<brief record id>",
    "fields": {
      "Status": "posted-as-drafts",
      "Posted To": [<the platforms that were sent>]
    }
  }]
)
```

If the user later flips the status to `live` after publishing in GHL,
that's a manual action — this skill does not own the `live` transition.

## Step 9: Hand-off

Output exactly this block (substitute placeholders):

```
✓ Drafts created in GHL Social Planner.

Brief: <Airtable record URL>
Status: posted-as-drafts
Mode: <draft | scheduled draft for <date>>

DRAFT LINKS:
- LinkedIn:  <draft URL from webhook response>
- Instagram: <draft URL from webhook response>

NEXT STEPS:

1. Open each draft in GHL.
2. Polish:
   - Check line breaks, spacing, video preview.
   - Add @-mentions, polls, or links if needed.
   - Crop the video thumbnail if you want a specific frame.
3. Publish (if mode is draft) or let the schedule fire (if scheduled).
4. After publishing, update the brief Status to `live` in Airtable.

If anything is wrong with the drafts, edit the brief in Airtable and
re-run /post-promo. (Note: this creates new drafts; delete the old ones
in GHL.)
```

## Important Rules

- NEVER fire the webhook without explicit user approval (Step 6 y/n).
- NEVER skip the caption lint (Step 5). The whole point of this pipeline
  is shipping captions that don't read as AI-generated; the lint is the
  last gate.
- NEVER auto-correct captions silently. Show the user the issues and
  let them decide.
- ALWAYS update Airtable after a successful webhook (so the brief
  shows `posted-as-drafts` and you don't accidentally double-send).
- The `live` status transition is manual. The user flips it after they
  publish in GHL.
- If the webhook fails, do NOT mark Airtable as `posted-as-drafts`. The
  status only advances when GHL actually has the drafts.
