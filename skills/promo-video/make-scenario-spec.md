# Make.com Scenario Spec: GHL Promo Posting

This Make scenario receives a webhook from `/post-promo` and creates
drafts (or scheduled drafts) in GoHighLevel's Social Planner across one
or more platforms.

## Naming

Suggested scenario name: **`RGA Promo → GHL Social Planner Drafts`**

## Trigger

**Module:** Webhooks → Custom webhook

**Webhook URL:** Copy this URL into the `MAKE_GHL_PROMO_WEBHOOK` env
var in both `ai-sales-agent-v5/.env` and
`rga-video-generator/.env`.

**Expected payload:**

```json
{
  "brief_id": "recerNJCKtxgjDGbY",
  "brief_name": "Meeting Prepper V2.0 — relaunch, whole-picture angle",
  "mode": "draft",
  "scheduled_at": null,
  "video_drive_url": "https://drive.google.com/file/d/<id>/view?usp=drivesdk",
  "video_drive_id": "1mmeyB3QncOfD-TryWT-pWAGew1f5HnE1",
  "platforms": ["LinkedIn", "Instagram"],
  "captions": {
    "LinkedIn": "<full caption text>",
    "Instagram": "<full caption text>"
  },
  "airtable_record_url": "https://airtable.com/app8WmO2rF14Bfv8O/.../recerNJCKtxgjDGbY"
}
```

`mode` is one of:
- `"draft"` — save as draft in GHL, no schedule.
- `"scheduled-draft"` — save with `scheduled_at` time set; user can edit
  before fire time.

## Steps

### 1. Receive webhook

Standard webhook trigger. Set sample data using the JSON above so the
mapper can resolve fields.

### 2. Download video from Drive

**Module:** Google Drive → Download a File

Use `video_drive_id` to fetch the binary. Map the returned file to the
next module.

(Alternative if your GHL plan supports it: pass `video_drive_url`
directly to GHL as a `mediaUrl`. Skip step 2 in that case. Most GHL
plans require uploading the binary first.)

### 3. Upload to GHL media library

**Module:** GoHighLevel → Upload Media

- **Location:** the RGA TechCXO sub-account (you have its ID; configure
  as a Make connection or hard-code in this step).
- **File:** the binary from step 2.
- **Filename:** derive from `brief_id` + `.mp4` (e.g.
  `recerNJCKtxgjDGbY.mp4`) so Make can dedupe on re-runs.

Capture the returned `mediaId` for use in step 4.

### 4. Iterator over platforms

**Module:** Iterator

Map `platforms` (array). Each iteration yields a single platform name
(`"LinkedIn"` or `"Instagram"`).

### 5. Create Social Planner post

**Module:** GoHighLevel → Create Social Post

For each platform iteration, create one post:

- **Location:** RGA TechCXO sub-account.
- **Platform / channel:** map from the iterator's current platform name
  to the corresponding GHL channel ID. Set up two channel connections
  in Make ahead of time, or use a switch module.
- **Post type:** Video.
- **Caption:** `captions[<platform>]` (map by current iteration).
- **Media:** `mediaId` from step 3.
- **Status / scheduled time:**
  - If `mode == "draft"`: set status to `Draft`. Leave scheduled time empty.
  - If `mode == "scheduled-draft"`: set status to `Scheduled`. Use
    `scheduled_at`.

Capture the returned post URL or ID per iteration.

### 6. Aggregate results

**Module:** Array Aggregator (collect from step 5)

Build a structure:

```json
{
  "LinkedIn": "<draft url>",
  "Instagram": "<draft url>"
}
```

### 7. Respond to webhook

**Module:** Webhooks → Webhook response

Status code: `200`
Body type: JSON

```json
{
  "ok": true,
  "drafts": {{ aggregated_object_from_step_6 }}
}
```

If any step in the iterator fails, respond with:

```json
{
  "ok": false,
  "error": "<error message>",
  "drafts": { /* partial drafts, if any */ }
}
```

Status code: `500` for full failure, `207` for partial.

### Error handling

- If Drive download fails: respond 500 with the error.
- If GHL media upload fails: respond 500.
- If a single platform's post creation fails but others succeed:
  respond 207 partial with the platforms that did succeed.

The skill (`SKILL-post.md`) only updates Airtable on `ok: true`. Partial
success means the user manually retries the failed platform.

## Optional follow-ups

These can be added to the scenario later, not required for v1:

- **Slack notification** when drafts are created (with brief name and
  draft links).
- **Calendar event** in Google Calendar at `scheduled_at` so you have a
  reminder to check the scheduled draft before it fires.
- **Airtable update from Make** (instead of from the skill) — but this
  duplicates work; let the skill own that update.

## Testing

1. Run `/post-promo <test-brief>` against a brief in `rendered` status.
2. Verify the webhook payload shape is what Make expects.
3. Verify drafts appear in GHL Social Planner.
4. Verify the brief's status flips to `posted-as-drafts` in Airtable.
5. Verify draft URLs come back in the chat hand-off.
6. Manually delete the test drafts in GHL.
