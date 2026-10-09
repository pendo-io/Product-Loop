# Setup

Run on first use, or partially when the PM asks to change a piece of their setup. Produces the `feature_validation` config (optional for a one-shot run). Ask closed-choice questions as short multiple choice; take free text (channel names, doc URLs) in prose. Always offer defaults so the PM can accept and move on.

## Step 1: Who's validating

Ask the PM's name if not obvious from context. If a tracker is connected, resolve their user in it (the value the owner field stores; in Notion this may be a page in a people database rather than a user id). If Slack is connected and they want run notices, resolve their Slack user. Store in `config.pm_user`.

## Step 2: Where features come from

Ask: *"Where should I get your features from? (a) a tracker I can read (Notion, Jira, Airtable, or similar), or (b) you'll paste a list each run."*

- **Tracker** → ask for the database / project / base URL, then introspect its schema (`references/tracker-setup.md`). Map each role in `config.fields` to the actual property, showing the defaults so the PM can confirm. Map the high-signal fields: keep the six defaults that exist, let the PM rename, add, or drop, and drop any the tracker doesn't have. Confirm the shipped phase label (default `GA`). Map the roadmap flag (`show_on_roadmap`, default `Show on customer-facing roadmap?`, any type: checkbox or select) along with the roadmap description; if the tracker has none, leave it null.
- **Pasted list** → show the expected shape (feature name, current value of each high-signal field, optional `Last Validated`, optional due flags, optional links to spec doc or ticket). Offer the default six fields and let the PM change them.

## Step 3: Due rule

- Tracker has due flags (default `Due for Validation`, `Pre-change due`) → use them.
- No due flags → ask for a validation cadence in days, suggesting 7 days (routine due when `Last Validated` is empty or older), and explain that a next-phase target date of today or tomorrow counts as pre-change due. Store in `config.due_rule`.
- Pasted list → every pasted feature is due unless the paste carries flags or dates.

In tracker mode, run the due query now and report: *"You own N features; M are due or pre-change right now."* This checks the query and sets expectations. Store nothing feature-specific; scope is recomputed every run.

## Step 4: Output

Default **markdown in chat**. Offer tracker write-back only if a tracker is connected and has (or the PM adds) `Last Validated` and `Validated by` (select, options `Agent`, `User`). Never create them. Store in `config.output.mode` (`markdown` or `tracker`).

## Step 5: Sources and the per-field map

If a weekly-update skill is installed with a saved source set, offer it as the default (accept / edit / start fresh). Don't force it.

Then gather, all optional:
- **Slack sources**: channels or DMs where launch announcements, EA chatter, and phase news appear.
- **Doc sources**: roadmap docs, GTM / launch brief folders, EA docs. Capture type (`notion_page`, `google_drive_folder`, `google_drive_file`, `confluence_space`, `confluence_page`, `url`) plus url and name.
- **Jira**: project key(s) the features live under, if known (narrows search).

With no connectors, the PM can paste evidence each run; say that fields without evidence will land in Tier B.

Present the **default per-field source map** (from `SKILL.md`) and let the PM adjust in prose, for example "access details are on our beta signup page, not in the spec." Store in `config.field_source_map`. Tell the PM they can change any source at any time just by saying so ("also watch #team-channel", "stop reading the old roadmap doc").

## Step 6: Notices and schedule (optional)

- **Notices** (only if Slack is connected): *"DM me when a run needs me"* (`mode: always`) / *"Only when something looks inaccurate"* (`only_if_tier_c`) / *"Don't DM, I'll check the chat"* (`slack: false`). Default: `always` if Slack is connected, otherwise off.
- **Schedule**: on demand by default. If the host supports scheduled tasks and the PM wants unattended runs, recommend **daily** and explain the silent-on-quiet-days behavior. Store in `config.schedule`.

## Step 7: Calibration (tracker write-back only)

Explain the one-time calibration: on the first write-back run, the agent shows its would-be auto-validations for the PM to sanity-check before committing any; after that first approval it runs autonomously. Default on (`calibration.enabled = true`, `calibration.done = false`). If declined, set `enabled = true, done = true` so the first run auto-validates immediately (still audit-commented, so it's recoverable). Recommend keeping it on.

## Step 8: Save

Offer to save. If yes, write the `feature_validation` section to `config.json` in the workspace folder (shape per `config.example.json`); if a daily-digest conductor keeps a shared config in the workspace, write the section there instead. Initialize an empty `state.json` (pending queue, coverage log) next to it. Don't commit a real config to a public repo; it holds user ids.

## Step 9: Schedule (if chosen)

Walk the PM through `references/scheduling.md`. The skill can't register the task itself. Offer a dry run ("run feature validation now") first.

## Re-setup (partial)

Map the PM's request to the relevant step and re-run just that:
- "change my sources" / "watch #x" / "stop reading <doc>" → Step 5.
- "validate different features" / "use a different tracker" → Step 2.
- "change my schedule" → Step 6, then Step 9.
- "change notifications" → Step 6.
- "re-detect my schema" (someone renamed a property) → Step 2.

Save back in place; don't rebuild the whole file.
