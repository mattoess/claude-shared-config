# Promo Video Post Procedure

This is the publish-side of the promo-video workflow. The brief was
created by `/promo-brief`, the video was built and rendered by
`/build-promo`, and now it's ready to ship to GoHighLevel.

You do **not** post directly to LinkedIn or Instagram. You send the post
to GoHighLevel's Social Planner as drafts (or pre-scheduled drafts if a
publish date is set). The user then polishes inside GHL — line breaks,
person tags, polls, video preview — and clicks publish.

The GHL Social Planner draft is created **directly via the GoHighLevel
MCP** (`mcp__ghl__social-media-posting_create-post`). No Make.com webhook
is involved. This skill validates the brief, lints the captions, shows a
preview, and on approval calls the GHL MCP once per platform.

> Prerequisite: the GoHighLevel MCP server must be connected in this
> session. If `mcp__ghl__*` tools are unavailable, halt and tell the user
> to enable the GHL MCP. (The MCP is location-scoped to the RGA GHL
> location; you do not pass a location id.)

## Working directory

You can run this from either repo:
- `~/Documents/ai-sales-agent-v5/` (where briefs are authored), or
- `~/Documents/Claude/rga-video-generator/` (where videos are built).

The skill is the same. Don't worry about which repo.

## Constants

- Base: `app8WmO2rF14Bfv8O` (RGA Promo Pipeline)
- Promo Briefs table: `tblrFA4SLyRcbQ965`
- Posting transport: GoHighLevel MCP (`mcp__ghl__social-media-posting_*`),
  location-scoped to the RGA GHL location. No env var or webhook needed.

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
**Accounts:** LinkedIn (RGA), Instagram (RGA)  (resolved from GHL in Step 7a)

---

### LinkedIn

[exact caption text as it will appear in GHL]

---

### Instagram

[exact caption text as it will appear in GHL]
```

Then ask: "Send these to GHL? (y/n)"

If no, stay in chat for further edits to Airtable before re-running.

## Step 7: Create the Drafts via GHL MCP

### Step 7a: Resolve social accounts (do this BEFORE the Step 6 preview)

Call `mcp__ghl__social-media-posting_get-account` to list the connected
social accounts for the location. The response contains accounts with an
`id` (the `accountId`) and a `platform`/`type` (e.g. `linkedin`,
`instagram`, `facebook`).

### Standing account selection (RGA default)

Unless the user says otherwise on the run, post to these specific
accounts. Match them by **name + platform** in the `get-account` results
(account IDs can change when an account is re-authed, so match by name
and read the current `id` from the response):

| Platform  | Account name           | Type    | Reference accountId (verify against get-account) |
|-----------|------------------------|---------|---------------------------------------------------|
| LinkedIn  | Matt Oess              | profile | `67ddd7fe838e587a959cab49_zTPFln9s8sc8hUWqgr4q_aW9gpZq5aK_profile` |
| LinkedIn  | Revenue Growth Agent   | page    | `67ddd7fe838e587a959cab49_zTPFln9s8sc8hUWqgr4q_107978108_page` |
| Instagram | mattoess               | profile | `67ddd7ba031a9629acb54252_zTPFln9s8sc8hUWqgr4q_17841402062630118` |

So **LinkedIn posts to BOTH** the Matt Oess profile and the Revenue
Growth Agent page; **Instagram posts to mattoess only**. (The LinkedIn
caption is the same text for both LinkedIn targets.)

Rules:
- Match each target by name (case-insensitive) and platform. If a named
  account is **missing** from `get-account`, halt and tell the user which
  one is gone (likely needs re-connecting in Social Planner).
- If `get-account` shows a name collision or an unexpected duplicate,
  list them and ask. Don't guess.
- Skip `community`, `youtube`, and group channels unless the user
  explicitly asks to post there.
- The user can override per-run ("just the RGA page", "skip Instagram",
  etc.). Honor the override.

Surface the resolved account names in the Step 6 preview ("Accounts:"
line) so the user confirms they're posting to the right handles.

### Step 7b: Create drafts, grouped by caption

On approval (Step 6 y/n), call `create-post` **once per distinct
caption**, passing all accountIds that share that caption in one call.
GHL's `create-post` takes an array of `accountIds`, so accounts on the
same platform that use the same caption go in a single call.

With the standing RGA selection that means **two** calls:

1. **LinkedIn** — both the Matt Oess profile and the Revenue Growth Agent
   page (same `LinkedIn Caption` text):
   ```
   body_accountIds = ["<Matt Oess LinkedIn id>", "<RGA page LinkedIn id>"]
   body_summary    = "<exact LinkedIn Caption text>"
   ```
2. **Instagram** — mattoess only (`Instagram Caption` text):
   ```
   body_accountIds = ["<mattoess IG id>"]
   body_summary    = "<exact Instagram Caption text>"
   ```

Full call shape:

```
mcp__ghl__social-media-posting_create-post(
  body_accountIds = [ ...ids sharing this caption... ],
  body_type       = "post",
  body_summary    = "<exact caption text for these accounts>",
  body_status     = "draft",          // see mode mapping below
  body_scheduleDate = "<ISO-8601>",   // ONLY when scheduled, else omit
  body_media      = [{ "url": "<Output MP4 URL>" }],
  body_userId     = "<GHL user id if known, else omit>"
)
```

Never mix two platforms' captions in one call. LinkedIn and Instagram
captions differ, so they are always separate calls even though both are
"post" type.

Field mapping:
- **`body_summary`**: the platform's exact caption from Airtable
  (`LinkedIn Caption` or `Instagram Caption`). Do not merge or alter.
- **`body_status`**:
  - mode `draft` → `"draft"` (omit `body_scheduleDate`).
  - mode `scheduled-draft` → `"scheduled"` and set `body_scheduleDate`
    to the ISO-8601 timestamp with timezone from Step 4. (GHL holds it
    as a scheduled post the user can still edit before it fires.)
- **`body_media`**: a single-element array referencing the rendered
  video at the brief's `Output MP4 URL`. (GHL fetches the media from the
  URL; the Vercel Blob / Drive URL must be publicly reachable.)
- **`body_type`**: `"post"` unless the brief explicitly calls for a
  `reel` or `story`.

Call it once per platform, collecting each response. Each successful
response returns the created post with an `id` and `status`.

### Step 7c: Handle failures

- If any platform's `create-post` call errors, halt immediately. Report
  which platforms succeeded and which failed. Do NOT mark Airtable as
  `posted-as-drafts` unless **all** requested platforms succeeded — a
  partial post would otherwise be silently lost on the next run.
- If the media URL is rejected (not publicly reachable), tell the user
  the `Output MP4 URL` must be public and halt.

## Step 8: Update Airtable

On success (all requested platforms posted in Step 7):

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

DRAFTS CREATED (open in GHL Social Planner):
- LinkedIn:  post id <id from create-post response>
- Instagram: post id <id from create-post response>

(GHL's create-post returns a post id, not a deep link. Find the drafts
in Social Planner → Drafts, newest first.)

NEXT STEPS:

1. Open each draft in GHL Social Planner.
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

- NEVER call `create-post` without explicit user approval (Step 6 y/n).
- ALWAYS create as a **draft** (or scheduled draft). NEVER post with
  `status: published` from this skill. The user publishes in GHL.
- NEVER skip the caption lint (Step 5). The whole point of this pipeline
  is shipping captions that don't read as AI-generated; the lint is the
  last gate.
- NEVER auto-correct captions silently. Show the user the issues and
  let them decide.
- ALWAYS update Airtable after ALL requested platforms post successfully
  (so the brief shows `posted-as-drafts` and you don't double-send).
- The `live` status transition is manual. The user flips it after they
  publish in GHL.
- If any platform's create-post fails, do NOT mark Airtable as
  `posted-as-drafts`. Report partial success and let the user re-run.
