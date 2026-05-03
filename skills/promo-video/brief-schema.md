# RGA Promo Brief Schema v1

The brief is the contract between `ai-sales-agent-v5` (drafts briefs) and `rga-video-generator` (builds videos). Stored across two `Promo Briefs` Airtable fields: `Script JSON` and `Shot List JSON`. Both must validate against this spec or the video build halts.

## `Script JSON`

```json
{
  "version": 1,
  "format": "square-20",
  "accent_color": "green",
  "scenes": {
    "problem": {
      "headline": "Most sales reps {accent}skip{/accent} prospect research.",
      "subtext": "Not because they don't care — because they can't find the basics.",
      "screenshot": "problem-empty-form"
    },
    "solution": {
      "headline": "RGA finds it for you. {accent}One click.{/accent}",
      "before_screenshot": "solution-button",
      "after_screenshot": "solution-enriched",
      "badges": ["Verified email", "LinkedIn", "Title", "Company details"]
    },
    "payoff": {
      "headline": "When reps have the basics, they {accent}actually do the research.{/accent}",
      "subtext": "And when they do the research, they win more deals.",
      "screenshot": "payoff-enriched"
    },
    "cta": {
      "headline": "Give your team the {accent}head start{/accent} they need.",
      "subtext": "Verified contact data. One click. Zero guesswork.",
      "button_text": "DM us for a demo"
    }
  }
}
```

### Field rules

- `version`: integer, currently `1`. Bump only with breaking schema changes.
- `format`: enum. See **Formats** below. Determines aspect ratio, total duration, and per-scene timing.
- `accent_color`: `"green"` or `"orange"`. Applied to all `{accent}...{/accent}` markers in the brief.
- `scenes`: must include all four keys: `problem`, `solution`, `payoff`, `cta`. Order is fixed.
- **All `headline` strings must contain exactly one `{accent}...{/accent}` pair.** This is the visual punch word in each scene. Validation rejects briefs without it.
- `subtext`: optional, plain string. Used in `problem`, `payoff`, `cta` scenes (not `solution`).
- `screenshot` / `before_screenshot` / `after_screenshot`: must be a `shot.id` from the Shot List. Validation cross-checks.
- `badges`: 4 strings max, ~3 words each. Solution scene only.
- `button_text`: ≤ 22 characters. Locked default: `"DM us for a demo"` (do not use the curly form). Only override with a documented reason.

## Captions

Drafted as part of the brief. Same iteration loop as the script. Stored
both as structured fields inside `Script JSON.captions` (for re-iteration)
and as pre-formatted long-text fields on the Airtable record (for
copy-paste).

```json
"captions": {
  "linkedin": {
    "hook": "Most reps walk into meetings half-prepared. We just fixed that.",
    "body": "Skimming LinkedIn five minutes before the call isn't research.\n\nMeeting Prepper V2.0 ships today. Type a name and a company; you walk in with pain points, talking points, discovery questions, and tailored solutions in 90 seconds.\n\nThe difference between a meeting and a deal is what you knew before it started.",
    "hashtags": ["B2BSales", "SalesEnablement", "SalesAI"],
    "cta": "DM me for a demo."
  },
  "instagram": {
    "caption": "Stop walking into meetings half-prepared.\n\nName. Company. One click. → Pain points, talking points, and discovery questions in 90 seconds.\n\nDM for a demo.",
    "hashtags": ["sales", "salestech", "b2bsales", "saleslife", "ai", "salesai", "revops", "prospecting"]
  }
}
```

### LinkedIn rules

- `hook`: first 1 to 2 lines. This is what's visible before "see more"
  truncates. Roughly ≤ 150 characters. The whole post lives or dies on this.
- `body`: 2 to 4 short paragraphs. Line breaks between them. Total post
  including hook + body should land in the 600 to 1300 character range.
  Longer is OK if the story warrants.
- `hashtags`: 3 to 5. PascalCase or single-word lower. No `#` prefix in
  the array; the skill adds it when concatenating.
- `cta`: one line. Default `"DM me for a demo."` — don't restate the brand,
  don't link out (LinkedIn deprioritizes posts with outbound links).

### Instagram rules

- `caption`: short. Single block or 2 short paragraphs. Aim ≤ 200
  characters before the hashtag block. The first line must hook because
  Instagram truncates aggressively in feed (~80 chars).
- `hashtags`: 5 to 10. Lowercase, single-word or compact compounds. The
  skill renders these as a trailing block separated by a blank line.

### Concatenation (what gets written to Airtable's caption fields)

**`LinkedIn Caption` field** (the long-text on the brief record) is the
pre-formatted post:

```
{hook}

{body}

{cta}

#{hashtag1} #{hashtag2} #{hashtag3}
```

**`Instagram Caption` field** is:

```
{caption}

#{hashtag1} #{hashtag2} ... #{hashtagN}
```

The structured form in `Script JSON.captions` is the source of truth;
the long-text fields are derived for fast copy-paste.

### Tone rules (apply to ALL captions)

- **No em-dashes (—).** Period. Use a comma, a colon, a period, or
  rewrite. Em-dashes are the most reliable AI-writing tell.
- **No "in today's fast-paced world", "game-changer", "leverage",
  "synergy", "robust", "seamlessly", "unlock", "elevate", "empower",
  "revolutionize", or other corporate filler.** Cut every word that
  isn't doing work.
- **No "we're excited / thrilled / proud to announce".** Just say
  what shipped.
- **No three-item lists used decoratively** ("faster, smarter, better").
  Lists are fine when they're real. Avoid the rhythmic AI cadence.
- **No rhetorical questions in the hook** ("Tired of X?"). Make a claim
  instead.
- **Active voice.** "Reps walk in with the whole picture," not "Reps
  are equipped with."
- **Never invent metrics.** "90 seconds" is OK if the code actually
  produces output in 90 seconds. "10x faster" is not OK unless someone
  measured it.
- **Match the script's tone.** If the script is direct and a little
  blunt, the captions are too. Don't pivot to LinkedIn-thinkfluencer
  voice.

If you find yourself writing a sentence that sounds like every other
LinkedIn post about AI tools, delete it.

### Formats

The `format` field collapses aspect ratio, total duration, and scene-timing recipe into a single value.

| format       | aspect    | dimensions  | duration | problem | solution | payoff | cta |
|--------------|-----------|-------------|----------|---------|----------|--------|-----|
| `square-20`  | 1:1       | 1080 × 1080 | 20s      | 5s      | 6s       | 5s     | 4s  |
| `square-30`  | 1:1       | 1080 × 1080 | 30s      | 7s      | 9s       | 8s     | 6s  |

Future formats (e.g. `vertical-20` at 1080×1920) can be added by extending the recipe table — no schema changes required.

## `Shot List JSON`

```json
{
  "version": 1,
  "shots": [
    {
      "id": "problem-empty-form",
      "description": "Empty contact form in CRM, before enrichment",
      "route": "/contacts/new",
      "viewport": "1080x1080 square selection",
      "state_setup": "Logged in as demo user, fresh contact form open",
      "what_to_capture": "Form with empty email/title/company fields visible",
      "filename": "problem-empty-form.png",
      "drive_url": null,
      "uploaded": false
    }
  ]
}
```

### Field rules per shot

- `id`: kebab-case, unique within the brief. Referenced by `Script JSON` scene fields.
- `description`: 1 sentence — what the shot shows.
- `route`: app route path where the shot is taken, or `n/a` for non-app shots.
- `viewport`: required; recommend `1080x1080 square selection` for consistency. If a non-square ratio is unavoidable, note it explicitly.
- `state_setup`: imperative steps to reach the state to capture ("Log in as X, navigate to Y, click Z"). Must be actionable by a human (or, eventually, Playwright).
- `what_to_capture`: exactly what should be visible/highlighted in the frame.
- `filename`: kebab-case, ends `.png`. Convention: `{id}.png`.
- `drive_url`: null until uploaded. Filled with the Google Drive shareable link.
- `uploaded`: false until the file is on Drive. The `/build-promo` command halts if any required shot has `uploaded: false`.

## Cross-validation rules

1. Every `screenshot` / `before_screenshot` / `after_screenshot` ID in `Script JSON` must exist as a `shot.id` in `Shot List JSON`.
2. Every shot in `Shot List JSON` should be referenced by at least one scene (otherwise it's dead weight).
3. All shots must have `uploaded: true` and a non-null `drive_url` before `/build-promo` proceeds.

## Accent marker conventions

The `{accent}...{/accent}` tag wraps the visually highlighted phrase in headlines. The video template parses it at render time and applies `accent_color` to the wrapped text.

Examples:
- `"Most sales reps {accent}skip{/accent} prospect research."` → "skip" colored
- `"RGA finds it for you. {accent}One click.{/accent}"` → "One click." colored

Rules:
- One pair per headline. (Not zero, not two.)
- Wrap 1–4 words. Longer phrases lose visual punch.
- Don't include trailing punctuation in the accent unless it visually belongs (e.g., `{accent}One click.{/accent}` is fine; `{accent}skip{/accent}.` keeps the period outside).

## Versioning

Bump `version` only when a change would break existing briefs. Today's v1 is locked. Examples that would bump to v2:
- Changing the four-scene structure (adding/removing/renaming required scenes)
- Changing accent marker syntax
- Changing field names or types of existing fields

Additive optional fields and new `format` enum values do **not** bump the version.
