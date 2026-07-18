Build a promo video from an approved brief in the RGA Promo Pipeline Airtable base.

Invoke the `promo-video` skill in build mode with arguments: $ARGUMENTS

If no arguments are provided, list recent Promo Briefs with status `screenshots-ready` and ask the user to pick one.

Build mode steps (overrides the default brief-creation flow):

1. Resolve the brief — by Airtable record id (`recXXX...`) or by name match.
2. Verify status is `screenshots-ready`. If not, halt and explain.
3. Parse and validate `Script JSON` and `Shot List JSON` against `skills/promo-video/brief-schema.md`.
4. For every shot, confirm `uploaded: true` and a non-null `drive_url`. If any are missing, list them and halt.
5. Download each screenshot from its `drive_url` to `public/{filename}`.
6. Generate a kebab-case `Composition ID` from the brief's feature slug + angle (e.g. `MeetingPrepperV2Relaunch`).
7. Add a new `<Composition>` entry to `src/Root.tsx` with the brief embedded as `defaultProps`. Place it after the existing entries.
8. Run `npx tsc --noEmit` to verify.
9. Update the Airtable brief: set `Status` to `in-production`, set `Composition ID` to the chosen ID.
10. Tell the user to run `npm run dev`, preview the new composition in Remotion Studio, and iterate. After they approve, render with `npx remotion render src/Root.tsx <CompositionID> out/<id>.mp4` and run the post-render hand-off.

Detailed instructions live in `skills/promo-video/SKILL-build.md`.
