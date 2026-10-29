---
name: product-lead-weekly-update
description: 'Drafts a Product Lead''s weekly rollup by synthesizing the weekly updates of the PMs on their team, then cross-checking those updates against the lead''s own meetings, Slack, and docs for added context or discrepancies. PM updates can come from the product-manager-weekly-update skill (a shared folder or tracker) or simply be pasted in. Outputs the rollup as markdown in chat by default, or writes it to your own tracker (a Notion database or similar) if one is connected and you ask. Use whenever a product lead or head of a product area asks for their "weekly rollup", "leadership update", "team update", or says "draft my rollup", "summarize my PMs this week", "how did my team do this week", or "what should I tell leadership this week". Works on demand or weekly. Single owner of the product lead''s weekly rollup and of flagging PM-vs-source discrepancies. Not for a single PM''s own update; use product-manager-weekly-update.'
license: MIT. LICENSE has complete terms
compatibility: Standalone. No required connectors or skills; with none connected it works from pasted PM updates. Reads PM updates saved by the product-manager-weekly-update skill if installed (shared folder or tracker). Uses these if connected, for the cross-check; a meeting-notes connector (Notion, Granola, or similar), Slack, Google Drive, Confluence/Jira, and a tracker (Notion or similar). Tracker output and scheduled runs are optional.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

# Product Lead Weekly Update

Drafts the product lead's weekly rollup by synthesizing the PM updates filed
by their team for the week, cross-checking against the lead's own meetings,
Slack, and docs for added context or discrepancies, and producing one
leadership-level update. "Product lead" means whoever manages a group of
PMs or product areas. "Leadership" means whoever reads the rollup (config
`audience.name`).

Sibling to `product-manager-weekly-update`: this skill consumes PM updates rather than
re-gathering from scratch. The sibling is optional. PM updates can be
pasted, read from a folder, or read from a tracker. If the user wants their
PMs to generate updates the same way and the sibling isn't installed, say it
exists (https://github.com/pendo-io/Product-Loop) and offer to install it:
in Claude Code, or anywhere you can write files and reach GitHub, offer to
do it yourself, and on the user's OK download that skill's folder from the
repo into `~/.claude/skills/product-manager-weekly-update/`. Elsewhere (for example the
Claude apps), tell the user to download the folder and upload it as a skill.
Never block on it.

## Done

- **Result:** one rollup titled `<Lead name> Update - <YYYY-MM-DD>` with a Summary and a body, as markdown in
  chat (plus a saved file when `output.markdown_folder` is set), or as one
  tracker record with its URL in tracker mode.
- **Done check:** every item in the drafting checklist in
  `references/update-format.md` passes (fixed title, Summary bullet count,
  length, discrepancy callout limit), and every PM in `team.pms` is either
  covered or handled per `style.absent_pm_handling`.
- **Next consumer:** the reader in `audience.name`, and the next run, which
  reads `last_run`.
- **Intent:** synthesize rather than re-narrate, and surface disagreement
  rather than smooth it over. When the steps don't cover a case, flag it
  for the lead instead of resolving it.

## Boundaries and authority

- **Never rewrite or edit PM updates**, because they stay authoritative on
  PM-level detail. Read them and write a new lead-level rollup; flag a
  disagreement instead of correcting it.
- **Never invent progress, dates, or links**, because every claim must
  trace to a PM update or a source read this run. If the sources are thin,
  the rollup is short.
- **Never create tracker properties or options, and never write computed,
  rollup, or auto-set fields**, because a tracker may accept a computed
  value and silently discard it. Keep unmatched content in the body and
  say so in chat (`references/tracker-mode.md`).
- **Don't enumerate a large Slack workspace**, because a workspace-wide
  search pulls in unrelated threads. Start from the configured channels
  and DMs.
- **Don't ask questions on scheduled runs**, because they are
  non-interactive. Use defaults for optional settings and note them in the
  draft. If something required is missing (no team and no update source),
  stop and leave a clear "setup incomplete" note (and a DM if Slack is
  connected and `user.slack_user_id` is saved; if the id is empty, say in
  the run's chat output that no DM was sent because no Slack user id is
  saved).
- **Leave out personal and people content**, because the rollup is read
  by leadership who shouldn't see it, even when the source discusses it:
  someone's health, family, or leave details beyond "out this week", compensation, performance feedback or reviews, hiring, HR
  matters, or access requests about a named person, and personal chat.
  Keep the work: decisions, progress, blockers, risks, asks. If one line
  mixes both, keep the work part and drop the rest.
- **Read back every write before reporting it done**, because a write can
  fail or drop a field without an error. Re-fetch the tracker record and
  check its title, product areas field, and body; re-read a saved
  rollup file or the `product_lead_weekly` config section; check the DM's
  returned permalink; and confirm an installed sibling's `SKILL.md` is on
  disk.

**Authority:** the skill can write (1) its own `product_lead_weekly`
config section (setup, the saved output preference, `last_run`); (2) the
rollup as a markdown file in `output.markdown_folder`; (3) one tracker
record per rollup, only when `output.mode` is `tracker` or the user asks;
(4) a Slack DM to the lead, for the review heads-up when
`review_step.notify` is `slack_dm` and for the scheduled "setup
incomplete" note; (5) the sibling skill's folder in
`~/.claude/skills/product-manager-weekly-update/`, only on the user's OK.
Everything else, including the PM updates, is a read.

## Judgment vs mechanics

The model judges the synthesis, which differences are substantive enough
to flag as discrepancies, and how to mirror the lead's prior style.
Connectors gather the PM updates and sources. The rest are fixed rules the
model still applies in prose, with no script behind them: the window
dates, the title, the Summary bullet count, the word range, the callout
limit, and the absent-PM set (`team.pms` minus the PMs whose updates were
found). Each run rebuilds from config and the sources at run time; don't
reuse a rollup or PM update list from an earlier run in the conversation.

## Honesty

- **A missing PM update is a signal, not a gap to fill.** Report absence
  per `style.absent_pm_handling`; never write a PM's week for them.
- **"Couldn't check" is not "didn't file."** Treat a PM as absent only
  after a query in a form known to work, because a wrong filter form
  returns zero rows silently (`references/tracker-mode.md`). Say which
  sources were skipped because they aren't connected.
- **A PM update is a claim.** Discrepancy flagging checks it against
  meetings, Slack, and the feature record, and phrases the difference as
  a question.
- **No time estimates** for the skill's own work.

## Modes

1. **Setup** (first run, optional). Gather the config in
   `references/setup.md` and offer to save it. Skip it if the user just
   wants a rollup now.
2. **Weekly run.** Gather the PM updates, supplement from the lead's own
   sources, synthesize, and output the rollup.

## Config

Settings live in the `product_lead_weekly` section of the shared config
file, by default `config.json` in the user's working folder. If the daily-
digest-conductor's shared config (`.daily-digest/config.json` in the
workspace) exists, use this skill's section of that file instead
(shape in `config.example.json`). `config.*` below means that section. The
sibling skills read the same file; leave their sections alone.

- **Config present** → weekly run.
- **Config absent** (or holding only a saved `output` preference) → ask whether to set up now (about 3 minutes) or just
  draft this week's rollup. If "just draft", ask only for what the run
  can't infer: the lead's name, the PMs on the team, and the PM updates (or
  where to find them). Never treat missing config as an error.
- **No writable folder** → run without saving and say that settings won't
  persist.

Re-setup triggers such as "update my Slack sources", "change my schedule",
"add a PM to my team", or "redo my setup" re-run only the matching step in
`references/setup.md`.

## Where PM updates come from

Resolve in this order, and use every one that's available:

1. **Pasted.** Any PM updates the user pastes in the chat. When nothing is
   configured, ask for them: "Paste your PMs' updates for the week, one
   after another."
2. **Folder.** `config.pm_updates.folder` (for example the shared
   `output.markdown_folder` the PMs' `product-manager-weekly-update` writes to). Read
   files dated from the window start up to the run date, so updates filed
   late (after the window ends) still count.
3. **Tracker.** `config.pm_updates.tracker_url`, the database the PMs'
   updates are written to. Query by author and created date from the
   window start up to the run date, so updates filed late still count
   (see `references/tracker-mode.md`).

Identify each update's PM and date from its title, which the
`product-manager-weekly-update` skill writes as `<PM name> - PM update - <YYYY-MM-DD>`,
and its features from the closing `**Features covered:**` line (or the
tracker's feature relation). For pasted updates in any other format, take
the PM and date from the text or ask once.

## Output

- **Default: markdown.** Produce the rollup in chat in the format in
  `references/update-format.md`: title, a Summary section, then the body.
- **Bring your own tracker (optional).** If `output.mode` is `tracker`, or
  the user asks, create the rollup as a new record in their tracker. Follow
  `references/tracker-mode.md`: read the schema first, map to existing
  properties by name, never create properties or options, and never write
  computed, rollup, or auto-set fields.
- **No saved output preference** (setup skipped or not done, so the
  section has no `output` settings). Show the rollup in chat, then ask
  once, right after it:
  > Where should this live?
  > 1. Chat only
  > 2. Save to a folder
  > 3. Write to your system of record (Notion, Confluence, Jira, or similar)

  Save the answer to `config.output` (create the section if needed and
  leave the rest of the config alone), and don't ask again on later runs.
  Chat only → `mode: markdown`. Folder → `mode: markdown` plus
  `markdown_folder` (ask for the path once, default `./weekly-updates/`),
  then save this rollup there. System of record → `mode: tracker`: ask for
  the destination URL once, validate it per `references/tracker-mode.md`
  (read the schema, never create properties), save it to
  `output.tracker`, then write this rollup there. Saving only `output`
  doesn't count as finished setup. If there's no writable folder, use the
  answer for this run and say it won't persist. Scheduled runs don't ask:
  they use markdown in chat and note in the draft that no output
  preference is saved. Setup's Step 8 (Output) is the full version of this
  question.

## Weekly run flow

1. **Read config** (if present): `user`, `audience`, `team`,
   `pm_updates`, `sources`, `output`, `template`, `style`, `review_step`,
   `schedule`.
2. **Determine the window.** First find the rollup day: under the
   `daily-digest` conductor it is the conductor's `cadence` for this skill
   (`weekly-<day>`) and `schedule.day` is not used; run alone, it is
   `schedule.day`. If neither is set, use the first working day of the
   lead's week and say in the draft's handoff note that it's the default. Run on the rollup day, the window
   is the 7 days before it (the rollup day minus 7 through the day
   before), in the lead's timezone. Run on any other day, use the 7 days
   ending today. Always use the actual current date.
3. **Gather PM updates**, the primary source.
   1. For each PM in `team.pms`, find their updates in the window (see
      "Where PM updates come from"). Look at run time, not earlier, so
      late filers are picked up: an update filed after the window ended
      but before this run counts for the week it reports on.
   2. Read each update in full, including the features it covers. The
      features list gives a built-in cross-link.
   3. Note any PM who **did not file**. Absence is a real signal; handle it
      per `config.style.absent_pm_handling`. In tracker mode, only treat a
      PM as absent after a query in a form known to work (see
      `references/tracker-mode.md`).
4. **Gather supplementary context**, the cross-check layer. Skip any
   source that isn't connected and say which. See `references/sources.md`.
   1. **Meeting notes** for the window. The query returns a list of
      notes; fetch each note to get its summary, and read the transcript
      only if the summary is missing.
   2. **Slack.** For each `sources.slack` entry, read the window. Filter
      hard: lead-authored, lead-mentioned, or naming one of the lead's
      product areas or a PM. Exception: for an entry with
      `read_all: true`, read every post in the window without that
      relevance filter (for small channels where anyone's post can be the
      week's signal). The personal-content stop in Boundaries still
      applies to every source, `read_all` channels included.
   3. **Watched docs.** Fetch each `sources.docs` entry; note edits and
      comments inside the window.
   4. **Prior rollups.** The lead's last 2 or 3 rollups (tracker, folder,
      or pasted), as style anchors for voice and body structure.
5. **Synthesize**, in this order of priority:
   1. PM updates are the **content backbone**. Don't re-narrate them
      verbatim; condense to theme-level progress and cross-cutting
      observations.
   2. Use supplementary context to **add color** (decisions made in the
      lead's meetings, escalations from Slack, customer signals from doc
      edits) and **flag discrepancies** (see below). Don't silently
      override a PM.
   3. Cover the five template prompts conceptually (headline, feature and
      initiative updates, customer commitments, risks and blockers, coming
      up), but format per the lead's own prior style. The customer
      commitments prompt applies only if the team tracks commitments
      (`config.template.tracks_customer_commitments`; missing means
      false); otherwise skip it and don't ask about commitments.
   4. Handle absent PMs per config. An absent PM goes in the Summary only
      when the absence hides a known risk (see
      `references/update-format.md` → Absent-PM handling).
6. **Assemble the rollup**: title, Summary (3 to 6 bullets), body. See
   `references/update-format.md`.
7. **Deliver.** Markdown in chat (and saved to `output.markdown_folder` if
   set), or, in tracker mode, create the record per
   `references/tracker-mode.md`.
8. **Review step.** In markdown mode the chat draft is the review; list
   any flagged discrepancies right above it in one line. In tracker mode,
   if `review_step.enabled`, tell the user in chat that the record is
   ready to edit in place, with "Discrepancies flagged: X, Y" if any. If
   `review_step.notify` is `slack_dm`, also DM them at
   `user.slack_user_id` (`references/sources.md` → Slack: send a DM); if
   Slack isn't connected or the DM fails, say so in chat. If
   `review_step.always_notify_on_discrepancy` is true, notify when
   discrepancies were flagged even if `enabled` is false.
9. **Record the run** in `config.last_run` (timestamp, status, output
   location, absent PMs, flagged discrepancies) if config is writable.

## Discrepancy flagging

The signature behavior of this skill: when the PM updates and the
supplementary context disagree, **surface it, don't paper over it**.
Examples:

- A PM reports a launch on track for the end of the month, but
  Thursday's meeting note says the customer asked to move it.
- A PM doesn't mention a customer escalation that's been the subject of a
  Slack thread the lead was tagged on.
- A PM says a milestone shipped but the linked feature record's status
  hasn't moved.

When you find one, add a one-line callout in the body, clearly marked:
`⚠️ Possible discrepancy: <PM> reports X but <meeting / Slack / feature>
suggests Y. Worth checking.` Don't try to resolve it; the lead is the human
in the loop. Record discrepancies in `last_run.flagged_discrepancies`.
Limits and rules are in `references/update-format.md`.

What to look for when comparing PM updates against the supplementary
context:

| Discrepancy type | How to detect |
| --- | --- |
| Date slipped silently | The PM update gives a target date; a meeting or Slack thread gives a different date for the same milestone |
| Customer escalation omitted | A Slack thread carries an escalation the PM didn't acknowledge in their update |
| Status mismatch | The PM says "shipped"; the linked feature record's status is still in progress |
| Resource change unmentioned | A meeting note mentions a team allocation change that isn't in the PM update |

Only flag a difference that is **substantive and recent** (within the
window). Phrase it as a question, not an assertion. The per-rollup limit
is in `references/update-format.md`.

## Asking the user for input

Use your client's multiple-choice question tool for closed choices
(confirm derived PMs, pick a schedule, style, absent-PM handling) if it has
one; otherwise show short numbered options. Ask free-text questions
(channels, doc URLs, a manual PM list) in prose.

## Gotchas

- **PM updates may not exist yet at run time** for PMs who file late.
  Read them at run time, and report absence.
- **Leads format differently.** Some write prose paragraphs grouped by a
  bolded initiative; some use bolded initiative headers (with or without
  emoji) and fixed-layout sub-bullets. Mirror the individual lead's prior
  body format, emoji or not. A first rollup with no prior format to mirror
  uses plain section headers with no emoji.
- **Title format is fixed.** Don't mirror prior titles (see
  `references/update-format.md`).
- **Derive the team from feature ownership, not a role field.** See
  `references/tracker-mode.md` → Deriving the team.
- **Filter by content, not by meeting or channel type.** 1:1s, standups,
  and small channels carry real work, so read them. The personal-content
  stop in Boundaries applies to every source, `read_all` channels
  included.
- **Skip bot and automation posts** (ticket-tracker notifications,
  deploy bots, integration alerts) unless the post carries a work signal
  for the week, such as a failed release, an incident, or a status change
  on a tracked feature.

## Scheduling (optional)

The skill works on demand. To run it weekly, register a scheduled task in
your client (for example `/schedule` in Claude Cowork or Claude Code) with
the prompt: "Run the product-lead-weekly-update skill. It reads its config
from config.json." If you use the `daily-digest` conductor, point the prompt
at `.daily-digest/config.json` instead, since that's where this skill's
section lives, and don't register a task or fill `schedule` (the
conductor's `cadence` holds the day). Default: early on your team's
rollup day (the first working day of the week if the lead doesn't name
one; say it's the default), in the lead's timezone, late enough to catch PMs who file late and early enough to land
before that day's meetings. Scheduled tasks generally only run while the
machine is awake and the app is open, so an early task may fire later if
the laptop was asleep; tell the lead during setup.

## Files in this skill

| File | When to load |
| --- | --- |
| `SKILL.md` | Always |
| `config.example.json` | Shape of the `product_lead_weekly` config section |
| `references/setup.md` | First run, or when the user changes part of the setup |
| `references/update-format.md` | When drafting |
| `references/sources.md` | When unsure how to read a source |
| `references/tracker-mode.md` | When reading PM updates from, or writing the rollup to, a tracker |
