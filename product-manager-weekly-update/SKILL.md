---
name: product-manager-weekly-update
description: 'Drafts a Product Manager''s weekly status update from meeting notes, watched Slack channels and DMs, watched docs, and owned features. Markdown in chat by default, or a record in your own tracker (Notion or similar) if connected and you ask. Use whenever the user asks for help with a "weekly update", "PM update", "weekly status", or asks to "draft my update", "write this week''s update", "summarize my week", "what should I tell my manager (or whoever reads it) this week", or "what''s gone on with my features this week". Works on demand or weekly. Single owner of a PM''s own weekly status draft; its output feeds product-lead-weekly-update. Not for rolling several PMs into one leadership update; use product-lead-weekly-update. Not for checking feature records are accurate; use feature-validation.'
license: MIT. LICENSE has complete terms
compatibility: Standalone. No required connectors or skills; with none connected it drafts from what the user pastes. Uses these if connected, to gather the week's context automatically; a meeting-notes connector (Notion, Granola, or similar), Slack, Google Drive, Confluence/Jira, and a feature tracker (Notion or similar). Tracker output and scheduled runs are optional.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

# Product Manager Weekly Update

Drafts the user's weekly PM update from sources they already produce (meeting
notes, Slack threads, watched docs, owned features), so the weekly update
takes no extra typing. "PM" means whoever owns the product decisions for a
set of features. "Manager" means whoever reads the update (config
`audience.name`).

Sibling skill, optional: `product-lead-weekly-update` rolls several PM
updates into one leadership update. It shares this skill's config file and
output conventions. If the sibling isn't installed and the user asks for its
job, say it exists (https://github.com/pendo-io/Product-Loop) and offer to
install it: in Claude Code, or anywhere you can write files and reach
GitHub, offer to do it yourself, and on the user's OK download that skill's
folder from the repo into `~/.claude/skills/<skill-name>/`. Elsewhere (for
example the Claude apps), tell the user to download the folder and upload
it as a skill. Never block on it.

## Done

- **Result:** one weekly update titled `<PM name> - PM update - <YYYY-MM-DD>`,
  as markdown in chat (plus a saved file when `output.markdown_folder` is
  set), or as one tracker record with its URL in tracker mode.
- **Done check:** every item in the drafting checklist in
  `references/update-format.md` passes: the title is exact, the Features
  covered line (or tracker relation) lists exactly the features in the
  body, and every claim traces to a source read this run. The user has
  the update or its location.
- **Next consumer:** the reader in `audience.name`, and the
  `product-lead-weekly-update` skill, which finds the update by its exact
  title and Features covered line.
- **Intent:** a short, true update. When the steps don't cover a case,
  leave a thing out rather than pad; a thin week makes a short update.

## Boundaries and authority

- **Never invent progress, dates, or links**, because every claim must
  trace to a source read this run. If a source is thin, write a shorter
  update.
- **Never copy a prior title**, because the product-lead skill finds PM
  updates by the exact title format. Mirror prior updates for the body
  only.
- **Never create tracker properties or options, and never write computed,
  rollup, or auto-set fields**, because a tracker may accept a computed
  value and silently discard it. Keep unmatched content in the body and
  say so in chat (`references/tracker-mode.md`).
- **Don't enumerate a large Slack workspace**, because a workspace-wide
  search pulls in unrelated threads. Start from the configured channels
  and DMs and search feature names within them; run a wider search only
  when the user asks.
- **Don't ask questions on scheduled runs**, because they are
  non-interactive. Use the default for a missing optional setting and note
  it in the draft. If something required is missing (no user name, no
  sources at all), stop and leave a clear "setup incomplete" note (and a
  DM if Slack is connected and `user.slack_user_id` is saved; if the id is
  empty, don't look it up unattended: say in the run's chat output that no
  DM was sent because no Slack user id is saved).
- **Leave out personal and people content**, because the update is read
  by people who shouldn't see it, even when the source discusses it: someone's health, family, or time off beyond "out this
  week", compensation, performance feedback or reviews, hiring or HR
  matters about a named person, and personal chat. Keep the work:
  decisions, progress, blockers, risks, asks. If one line mixes both, keep
  the work part and drop the rest.
- **Ask before overwriting** a saved update file, because the user may
  have edited it since; a scheduled run adds `-2` instead.
- **Read back every write before reporting it done**, because a write can
  fail or drop a field without an error. Re-fetch the tracker record and
  check its title, features field, and body; re-read a saved update
  file or the `pm_weekly` config section; check the DM's returned
  permalink; and confirm an installed sibling's `SKILL.md` is on disk.

**Authority:** the skill can write (1) its own `pm_weekly` config section
(setup, the saved output preference, `last_run`); (2) the update as a
markdown file in `output.markdown_folder`; (3) one tracker record per
update, only when `output.mode` is `tracker` or the user asks; (4) a Slack
DM to the user, for the review heads-up when `review_step.notify` is
`slack_dm` and for the scheduled "setup incomplete" note; (5) the sibling
skill's folder in `~/.claude/skills/`, only on the user's OK. Everything
else is a read.

## Judgment vs mechanics

The model judges what was substantive this week, the phrasing, which body
pattern (A or B) matches the PM's prior updates, and the voice. Connectors
gather the sources. The rest are fixed rules the model still applies in
prose, with no script behind them: the window dates, the file name, the
title, the match between the Features covered line and the body, and the
150 to 500 word length. Each run rebuilds from config and the sources;
don't reuse a draft from an earlier run in the conversation.

## Honesty

- **An empty week is a valid result.** Omit features with nothing
  substantive; no "no update this week" filler
  (`references/update-format.md` → Content coverage).
- **"Couldn't check" is not "nothing happened."** Say which sources were
  skipped because they aren't connected. In tracker mode, a wrong filter
  form returns zero rows that look like "no prior updates"
  (`references/tracker-mode.md`).
- **Unknown is not empty.** The "no metrics set" rule is in
  `references/update-format.md`.
- **No time estimates** for the skill's own work.

## Modes

1. **Setup** (first run, optional). Gather the config in
   `references/setup.md` and offer to save it. Skip it if the user just
   wants a draft now.
2. **Weekly run.** Read the configured sources (or what the user pastes),
   synthesize, and output the update.

## Config

Settings live in the `pm_weekly` section of a shared config file, by
default `config.json` in the user's working folder. If the `daily-digest`
conductor's shared config (`.daily-digest/config.json` in the workspace)
exists, use this skill's section of that file instead (shape in
`config.example.json`). `config.*` below means that section. The sibling
skills read the same file, so keep other sections untouched.

- **Config present** → weekly run.
- **Config absent** (or holding only a saved `output` preference) → ask whether to set up now (about 3 minutes) or just
  draft this week's update. If "just draft", ask only for what the run
  can't infer: the user's name, the features to cover (or "whatever I
  paste"), and any context they want to paste. Never treat missing config
  as an error.
- **No writable folder** (for example a chat client without file access)
  → run without saving and say that settings won't persist.

Re-setup triggers such as "update my Slack sources", "change my schedule",
"add a feature I own", or "redo my setup" re-run only the matching step in
`references/setup.md`.

## Output

- **Default: markdown.** Produce the update in chat in the format in
  `references/update-format.md`. If the config has
  `output.markdown_folder`, also save it there as
  `<YYYY-MM-DD>-pm-update-<pm-name-slug>.md` (name lowercased, spaces as
  hyphens) so the product-lead skill can read it. If that file already
  exists, ask before overwriting (a scheduled run adds `-2`). Save only
  the update itself; handoff notes such as the style-fallback note go in
  chat, never in the file.
- **Bring your own tracker (optional).** If `output.mode` is `tracker`, or
  the user asks, write the update as a new page or record in their tracker.
  Follow `references/tracker-mode.md`: read the target's schema first, map
  to existing properties by name, never create properties or options, and
  never write computed, rollup, or auto-set fields.
- **No saved output preference** (setup skipped or not done, so the
  section has no `output` settings). Show the draft in chat, then ask once,
  right after it:
  > Where should this live?
  > 1. Chat only
  > 2. Save to a folder
  > 3. Write to your system of record (Notion, Confluence, Jira, or similar)

  Save the answer to `config.output` (create the section if needed and
  leave the rest of the config alone), and don't ask again on later runs.
  Chat only → `mode: markdown`. Folder → `mode: markdown` plus
  `markdown_folder` (ask for the path once, default `./weekly-updates/`),
  then save this draft there. System of record → `mode: tracker`: ask for
  the destination URL once, validate it per `references/tracker-mode.md`
  (read the schema, never create properties), save it to
  `output.tracker`, then write this update there. Saving only `output`
  doesn't count as finished setup. If there's no writable folder, use the
  answer for this run and say it won't persist. Scheduled runs don't ask:
  they use markdown in chat and note in the draft that no output
  preference is saved. Setup's Step 7 is the full version of this
  question.

## Weekly run flow

1. **Read config** (if present): `user`, `audience`, `sources`,
   `features`, `output`, `template`, `style`, `review_step`, `schedule`,
   `last_run`.
2. **Determine the window.** It runs from your last update to today, in
   the user's timezone: from the day after `last_run.iso_timestamp` when
   that run's status is `delivered` or `published`, or the last 7 days
   ending today if there's no earlier update. Always use the actual
   current date; a delayed scheduled run still ends the window today.
   **Update day:** under the `daily-digest` conductor, the day is the
   conductor's `cadence` for this skill (`weekly-<day>`) and
   `schedule.day` is not used; run alone, it is `schedule.day`. If
   neither is set, use the last working day of the user's week and
   say in chat that it's the default.
3. **Gather context**, in this order, because later steps refer to earlier
   ones. Skip any source that isn't connected and say which ones you
   skipped. See `references/sources.md` for tool patterns.
   1. **Owned features.** From the connected feature tracker (filter on
      the owner field set in `features.owner_field`), or
      `features.list` in config, or ask the user to list them.
   2. **Meeting notes.** From the meeting-notes connector, for the window.
      The query returns a list of notes; fetch each note to get its
      summary, and read the transcript only if the summary is missing.
      Summarize long ones before drafting.
   3. **Slack.** For each `sources.slack` entry, read the window. Keep
      threads the PM wrote, was mentioned in, or that name one of the
      owned features. Feature-name searches are scoped to the configured
      and watched channels, not the whole workspace, unless the user asks
      for a wider search; the same keep rule and content filter apply to
      search results.
   4. **Watched docs.** Fetch each `sources.docs` entry. Note edits and
      comments inside the window.
   5. **Pasted context.** Anything the user pastes counts as a source,
      and is the only source when nothing is connected. In an on-demand
      run where no source is configured or connected, ask the user to
      paste their week (meeting notes, Slack threads, doc changes) before
      drafting.
   6. **Prior updates.** The PM's last 2 or 3 updates, from the tracker
      in tracker mode, from `output.markdown_folder`, or pasted. These
      are style anchors: match the PM's voice and body structure. The
      title format is fixed regardless (see `references/update-format.md`);
      mirror only the body, never a prior title.
4. **Draft the update** per `references/update-format.md`. Each feature
   gets a bullet that leads with the feature name (a link to the feature
   record if one exists), then 1 to 3 sentences on progress, blockers,
   decisions, and what's next. The empty-week, customer-feedback, and
   "no metrics set" rules live in `references/update-format.md`. Include
   "Additional work" for items that don't map to a tracked feature.
5. **Deliver.** Markdown in chat (and saved to `output.markdown_folder` if
   set), or, in tracker mode, create the record and link it to every
   feature the body covers.
6. **Review step.** In markdown mode the chat draft is the review. In
   tracker mode, if `review_step.enabled`, tell the user in chat that the
   record is created and ready to edit in place. If `review_step.notify`
   is `slack_dm`, also DM them at `user.slack_user_id`
   (`references/sources.md` → Slack: send a DM); if Slack isn't connected
   or the DM fails, say so in chat. Trackers like Notion have no draft
   state, so review means publish, then edit.
7. **Record the run** in `config.last_run` (timestamp, status, output
   location) if config is writable.

## Asking the user for input

Use your client's multiple-choice question tool for closed choices (confirm
inferred features, pick a schedule, pick a style) if it has one; otherwise
show short numbered options. Ask free-text questions (channel names, doc
URLs, style notes) in prose. Don't list options in prose and also show them
as buttons.

## Gotchas

- **The feature list in the update mirrors the body, not ownership.** Link
  or list only the features the body covers this week, never the full
  owned list.
- **PMs format updates differently.** Some group by product area with
  feature sub-bullets, some write one paragraph per feature. Mirror the
  individual PM's prior body format (see `references/update-format.md`).
- **Filter by content, not by meeting type.** 1:1s and standups carry
  real work, so read them. The personal-content stop is in Boundaries.

## Scheduling (optional)

The skill works on demand. To run it weekly, register a scheduled task in
whatever your client offers (for example `/schedule` in Claude Cowork or
Claude Code) with the prompt: "Run the product-manager-weekly-update skill. It reads its
config from its section of config.json (see Config)." If you use the
`daily-digest` conductor, point the prompt at `.daily-digest/config.json`
instead, since that's where this skill's section lives. Default: late in
the day on your team's update day, in the user's timezone, leaving time to
review before the day ends; if the user doesn't name a day, use the last
working day of their week and tell them it's the default. Keep
`config.schedule` in sync with what was registered. Under the conductor,
don't register a task or fill `schedule`: the conductor's `cadence` holds
the day (see Weekly run flow step 2).
Scheduled tasks generally only run while the machine is awake and the app
is open, and a connector that needs re-authentication will fail the run;
tell the user both during setup.

## Files in this skill

| File | When to load |
| --- | --- |
| `SKILL.md` | Always |
| `config.example.json` | Shape of the `pm_weekly` config section |
| `references/setup.md` | First run, or when the user changes part of the setup |
| `references/update-format.md` | When drafting |
| `references/sources.md` | When unsure how to read a source |
| `references/tracker-mode.md` | Only in tracker mode |
