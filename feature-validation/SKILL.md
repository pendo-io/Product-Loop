---
name: feature-validation
description: 'Keeps a product manager''s feature records accurate by validating the ones that are due. It takes features from your own tracker (Notion, Jira, Airtable, or similar) or a pasted list, researches each one''s high-signal fields against your sources, and acts by confidence tier: it marks as validated the ones it can positively corroborate (validated by Agent), and brings you the uncertain or inaccurate ones to resolve (validated by User). Markdown report in chat by default; tracker write-back is opt-in. Runs on demand or on an optional daily schedule. Single owner of feature-record validation stamps (Last Validated, Validated by). Use whenever the user asks to "validate my features", "check what''s due for validation", "revalidate", "are my features accurate", "check my roadmap accuracy", "run feature validation", or anything about confirming feature records are still true. Not for creating feature records or writing roadmap descriptions; use feature-creation.'
license: MIT. LICENSE has complete terms
compatibility: Standalone. Works with a pasted feature list and pasted evidence, no connectors needed. Optional connectors used if present, a tracker (Notion, Jira, Airtable, or similar) for the feature list and write-back, Jira, Slack, Google Drive, Confluence, and Notion for evidence, and Slack for a run notice. The feature-creation skill, if installed, drafts missing roadmap descriptions. Scheduled runs need a host with scheduled tasks (for example Claude Cowork).
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

## When to use

Use whenever the user asks to "validate my features", "check what's due for validation", "revalidate", "are my features accurate", "check my roadmap accuracy", "run feature validation", or anything about confirming feature records are still true. Also runs unattended on a schedule if the user set one up.

# Feature Validation

Keeps the feature records honest without making the PM babysit them. The tracker (or the PM) decides *which* features are due. This skill does the part a due-date reminder can't: it actually **checks each due feature against the PM's sources**, then either clears it or brings the PM exactly the ones that need a human, nothing more.

"PM" below means whoever owns the feature records being validated.

The governing principle, and the reason this is safe to run unattended: **absence of evidence is never confidence.** The agent auto-validates only what it can *positively corroborate* against recent, authoritative signal. Silence, staleness, or "found nothing contradicting it" all route to the PM, never to auto-validation. And auto-mode can only *confirm* a record; it never edits a field value on its own. Any change goes through the PM.

This skill has two modes and, within a run, two phases:

1. **Setup** (first run, optional to save): pick where features come from, map the tracker's fields, choose sources and output. See `references/setup.md`. A one-shot run can skip saving anything.
2. **Validation run**, two phases in one chat:
   - **Autonomous phase** (unattended-safe): find due features, research each, auto-validate Tier A, stage Tier B/C.
   - **Interactive phase** (when the PM is present or says "go"): walk the staged Tier B/C items with the PM one at a time.

## Done

- **Result:** a validation report in chat. Every due feature sits in exactly one tier: Tier A (validated by Agent, with its audit line), Tier C (proposed edits with evidence), or Tier B (confirm as-is). In tracker mode, Tier A stamps are written to the tracker.
- **Done check:** every due feature appears in exactly one tier with its audit line or staged values; shipped and other PMs' features appear nowhere; if config is saved, `last_run` holds the timestamp, per-tier counts, and status, and `state.json.pending_resolution` matches the Tier B/C queue shown in chat. Phase 2 is done when the queue is empty and `pending_resolution` is cleared.
- **Next consumer:** the PM, in this chat. If the `daily-digest` conductor runs this skill, it reads the tiers (Tier A as done, Tier B/C and the backfill offer as needing the PM's OK). A later session resumes the queue from `state.json`, not from this conversation.
- **Intent:** when the steps don't cover a case, protect the trustworthiness of the feature records. Absence of evidence is never confidence; when torn between Tier A and Tier B, choose Tier B.

## Boundaries and authority

- **Never auto-validate on absence of evidence,** because the whole safety model rests on positive corroboration; route silence, staleness, or "nothing contradicts it" to Tier B instead.
- **Never edit a field value in auto-mode,** because this caps the worst case at "validated something it shouldn't have," never "silently rewrote the roadmap"; stage the change as Tier C for the PM.
- **Never research, stamp, or edit a shipped feature** (phase = `config.fields.shipped_phase`), because release notes and launch tracking own a feature once it ships, and re-stamping would overwrite the record of what shipped; drop it before research.
- **Never touch another PM's features,** and re-check ownership before every write, because another PM's record is theirs to vouch for; skip the feature, write nothing, and say it isn't theirs to validate.
- **Never create tracker properties or options,** because a mismatch can silently create a duplicate option in some trackers; report what you would have written and tell the PM which property to add.
- **Never write computed fields** (formulas, rollups, lookups); corroborate a rollup against its source records' own dates instead.
- **Never ask interactive questions in the autonomous phase,** because nobody may be there; present the queue as plain, resumable prose and end the turn.
- **Never validate a Tier C feature** until both the agent and the PM are satisfied it's accurate; keep working the fields with the PM.

**Authority:** invoking this skill permits these writes, and no others:
1. Tracker (opt-in tracker mode only): `Last Validated` and `Validated by` on Tier A after calibration, and on Tier B/C after the PM's go-ahead; Tier C field edits the PM agreed; an audit comment (or a PM-named notes field). Re-fetch the record after each write and confirm the stamp and any edited values before reporting it validated; a write that doesn't read back is reported as unconfirmed.
2. This skill's config: the `feature_validation` section of `config.json` (or of the conductor's `.daily-digest/config.json`), at setup and on each run (`last_run`, `resolved_property_names`, `due_rule.cadence_days`, `calibration.done`). Re-read the file after each write.
3. `state.json` (`pending_resolution`, `coverage_log`, coverage counters, declined insights), only if a state file is kept. Re-read it after each write.
4. A Slack DM to the PM (opt-in `notify.slack`), including the auth-failure notice; confirm it by opening the returned message permalink.
5. Installing feature-creation into `~/.claude/skills/feature-creation/`, only on the user's OK.
6. Handing off to feature-creation's Backfill mode, whose writes that skill gates.

## Judgment vs mechanics

The model judges: whether each field is corroborated, uncorroborated, or contradicted; recency and authority of evidence; the tier (the 95% bar is a judgment, not a computed score); the proposed Tier C edits; and phrasing. The tracker owns the due flags when it has them; filter on them, don't reconstruct them. Everything else is still done in prose by the model, with no script: the cadence due rule when the tracker has no flags, the shipped and ownership re-checks before writes, the coverage counters, ratio threshold and log rotation, notify gating, and audit-line formatting. Follow those rules exactly as written in `references/`.

## Honesty

- **Empty is valid.** Most days nothing is due; exit silently with `last_run.status = "skipped_none_due"`.
- **Couldn't check is not nothing there.** A connector that needs re-auth gets `failed_auth` and its own notice, never a silent retry; a source that isn't connected leaves its fields uncorroborated, and a field missing from a paste shows as "not provided". Never guess a value.
- **Small samples are labelled.** Coverage insights wait for the thresholds in `references/sources.md`; a pattern from a single run is framed as an observation.
- **No time estimates** for the agent's own work; describe scope instead.

## Output modes

- **Default: markdown in chat.** The run produces a validation report: Tier A features with their audit lines, then the Tier C and Tier B queue. Nothing is written to your tracker; the audit lines are ready to paste into the feature records. If config is saved, the run still updates this skill's own `config.json` (`last_run`) and `state.json` (see Authority below).
- **Bring your own tracker (opt-in).** If the PM asks for write-back and a tracker is connected, the skill writes the validation stamp (`Last Validated`, `Validated by`) and, for Tier C, the PM-agreed field edits, plus an audit comment. It reads the tracker's schema first, uses its exact property names and option labels, and **never creates properties or options.** If a property it needs is missing, it lists what it would have written in chat instead. See `references/tracker-setup.md`.

## Step 0a: Environment check

- If the host supports scheduled tasks (for example Claude Cowork) and the PM wants unattended runs, setup ends by walking them through registering one (`references/scheduling.md`).
- Otherwise, or if the PM just wants a check now, run on demand. Scheduling is optional; the on-demand run is the same flow.

## Step 0b: Config check

Look for this skill's config: the `feature_validation` section of `config.json` in the connected workspace folder (if the `daily-digest` conductor manages a shared config at `<workspace>/.daily-digest/config.json`, use that file's `feature_validation` section). Throughout this skill, `config.*` means that section.

- **Config present** → skip to *Validation run flow*.
- **Absent** → follow `references/setup.md`. For a one-shot run, collect only what this run needs (feature list, sources) and offer to save it at the end.

Re-setup triggers: *"change my sources"*, *"validate different features"*, *"change my schedule"*, *"redo setup"*, *"stop checking <doc>"*, *"also watch #<channel>"* → re-run the relevant `references/setup.md` section and save back in place. Source edits can happen by prompting at any time, not only during setup.

**Companion skills (optional).** If a weekly-update skill is installed and has a saved source set, setup offers to reuse it. If a daily-digest conductor is installed, it can run this skill on its cadence. Neither is required. If the PM wants them and they aren't installed, mention they exist at https://github.com/pendo-io/Product-Loop and offer to install; never block on it.

---

## Where the feature list comes from

Pick one at setup (`config.feature_source.type`):

- **Tracker** (`tracker`): a Notion database, Jira project, Airtable base, or similar. The skill queries the PM's own features that are due (see step 3). Property names come from `config.fields`.
- **Pasted list** (`pasted`): the PM pastes features (a table, CSV, or bullets) with the current values of the high-signal fields. Every pasted feature is treated as due, unless the PM includes due flags or `Last Validated` dates, in which case the due rule below applies. The PM supplying the list is taken as ownership. A high-signal field missing from the paste is shown as "not provided" and counts as uncorroborated; never guess its value. Output is markdown only.

## Validation run flow

### Phase 1: Autonomous (runs whether or not the PM is present)

1. **Read config** → `pm_user`, `feature_source`, `fields`, `high_signal_fields`, `due_rule`, `sources`, `field_source_map`, `output`, `schedule`, `calibration`, `notify`, `coverage`, `last_run`. Read `state.json` if present (pending queue, coverage log). For a one-shot run without config, use the defaults in `config.example.json` and what the PM gave you.
2. **Determine today's date** from the system clock; never assume. Scheduled runs may fire late.
3. **Find due features.** Only the invoking user's own features: those where the PM is the owner (`config.fields.owner`, default `Product Manager`) that are due by `config.due_rule`:
   - **Tracker due flags (default).** If the tracker has due flags (default names `Due for Validation` for routine cadence elapsed, and `Pre-change due` for an imminent scheduled phase change), select features where **either** is true. Let the tracker own that logic; don't reconstruct it. `Pre-change due` catches phase changes scheduled for today or tomorrow, so it must be included, not just the routine cadence.
   - **No due flags in the tracker.** Use `due_rule.cadence_days` (setup asks for it). If it isn't set, don't ask here: use 7 days for this run, say so in the summary, and stage the question for Phase 2 ("Validation cadence: keep 7 days or choose another?"), saving the answer as `due_rule.cadence_days`. A feature is due when `Last Validated` is empty or older than that cadence (routine), or when its next-phase target date is today or tomorrow (pre-change). If the tracker has no `Last Validated` either, every owned feature in scope is due.
   - **Pasted list.** All pasted features are due unless the paste carries due flags or dates.
   Two hard scope rules:
   - **Exclude shipped features.** Skip any feature whose current phase is the shipped phase (`config.fields.shipped_phase`, default `GA`). Drop it before research; never research, stamp, or edit it.
   - **Own features only; never touch another PM's.** Only features where the invoking user is the listed owner are ever queried, and the write-gate re-checks ownership before any write.
   - **If nothing is due → exit silently.** Write `last_run.status = "skipped_none_due"`. No notice, no chat noise. This is the common case.
4. **For each due feature**, run the research and scoring procedure in `references/validation-logic.md`:
   - Read the current values of the `high_signal_fields`, and note whether the roadmap description (`config.fields.roadmap_description`) is empty (see "Empty roadmap descriptions"). This doesn't affect the tier.
   - Gather evidence per `field_source_map`, **recency-weighted** (most recent authoritative signal wins; stale signal counts as weak). With no connectors, use evidence the PM pasted or linked; otherwise the field is uncorroborated.
   - Classify each field: **corroborated** / **uncorroborated** / **contradicted**.
   - Assign a tier:
     - **Any field contradicted, or the agent believes any field should change → Tier C.** (Overrides confidence. A believed inaccuracy always lands here.)
     - **No change believed necessary AND all high-signal fields corroborated AND overall ≥95% → Tier A.** The 95% is a judgment bar, not a computed score; don't calculate a number (see `references/validation-logic.md`).
     - **Otherwise → Tier B.**
   - Feature confidence is **gated by its weakest field**; one shaky field drops the whole feature out of Tier A.
   - If a state file is kept, log per-field outcomes to `state.json.coverage_log` (see `references/sources.md`).
5. **Act on Tier A: auto-validate.** *Unless this is the calibration run* (step 7). For each Tier A feature: **first re-confirm the invoking user is the feature's owner; if not, skip it, write nothing, and note it.**
   - **Markdown mode:** list it as validated by Agent with its audit line.
   - **Tracker mode:** set `Last Validated` = today and `Validated by` = `Agent`, and add an audit comment citing exactly what corroborated each field (format in `references/validation-logic.md`).
   Auto-mode stamps validation only; it never edits a field value.
6. **Stage Tier B and Tier C; write nothing yet.** For each, capture the current values, the per-field corroboration result, and (Tier C only) drafted proposed edits with rationale and source. Write these to `state.json.pending_resolution` if a state file is kept; otherwise hold them in the chat.
7. **Calibration gate (first tracker-write run only).** If output is tracker mode and `calibration.done == false`, do **not** auto-write Tier A. Present the would-be Tier A decisions (with their audit evidence) for a one-time sanity check, alongside the Tier B/C queue. Only after the PM approves, write those validations and set `calibration.done = true`. Every run after that auto-writes Tier A without asking. (Markdown mode writes nothing, so it needs no calibration.)
8. **Hand off.** End Phase 1 with:
   - A concise chat summary: *"Validated N. M need your confirmation. K need a revision. Say 'go' to work through them."* List pre-change items first. (On the calibration run, frame the Tier A block as "would auto-validate, approve?")
   - If any due feature has an empty roadmap description, one line listing them and the backfill offer (see "Empty roadmap descriptions"), noting when the roadmap flag isn't mapped so off-roadmap features couldn't be skipped.
   - If `notify.slack` is on and Slack is connected, a DM pointing the PM to this chat (see `references/sources.md` → "Notify"). Respect `notify.mode` (`always` / `only_if_tier_c` / `never`).
   - `last_run` written (if config is saved): timestamp, counts per tier, status.
   - **Don't ask interactive questions during the autonomous phase**; nobody may be there. Present the queue as plain, resumable prose and end the turn.

### Phase 2: Interactive (when the PM is present or says "go")

Load the pending queue. **Ownership gate: only the feature's own PM may resolve its items. Before writing any Tier B confirmation or Tier C edit, re-confirm the invoking user is the feature's owner; if not, write nothing and say it isn't theirs to validate.** Walk items **one at a time**, Tier C first, then Tier B. For each:

- **Tier B: confirm as-is.** Show the feature's current values for the high-signal fields exactly as they stand. Add a short line naming which field(s) the agent *couldn't* corroborate, **but propose no changes.** Ask: *"Still accurate?"*
  - PM confirms → validated by `User` (tracker mode: set `Last Validated` = today, `Validated by` = `User`, add an audit comment; markdown mode: give the audit line). Remove from queue.
  - PM says something's off → convert to Tier C on the spot.
- **Tier C: revise then validate.** Show the current values, the field(s) the agent believes are wrong, the proposed new value(s), and the evidence. Work with the PM until every field is right. Then offer to validate. On the PM's go-ahead → tracker mode: apply the field edits, set `Last Validated` = today, `Validated by` = `User`, add an audit comment summarizing what changed and why; markdown mode: give the corrected values and the audit line. Remove from queue.
- Never validate a Tier C feature until both the agent and the PM are satisfied it's accurate.

After the queue empties, update `state.json` (clear `pending_resolution`) and, if any coverage pattern has crossed threshold, surface **one** coverage insight (see `references/sources.md` → "Coverage insights"). If the hand-off listed empty roadmap descriptions and the PM hasn't answered the offer, repeat it once. Then stop; don't invent follow-on work.

Use plain conversational confirmation in Phase 2 ("reply yes to validate, or tell me what's off"), not buttons; the walk-through is sequential and free-form.

## Empty roadmap descriptions (optional hand-off)

While reading each due feature, check its roadmap description field. Skip the check if the field isn't mapped (`config.fields.roadmap_description` is null) or a pasted list doesn't include it; never guess. Also skip features that aren't on the roadmap, since they don't need a roadmap description: a feature is off the roadmap only when its roadmap flag (`config.fields.show_on_roadmap`, a checkbox or a select) reads No (an unchecked checkbox, or the select's No option). If the flag isn't mapped (null, or a pasted list doesn't include it), skip nothing: run the check on every due feature and say in the report that the roadmap flag isn't mapped. An empty select or any other value also counts as on the roadmap. The check never changes a tier, and this skill never drafts or writes the description itself. Validation always completes whether or not the hand-off happens.

- **feature-creation installed:** list the features with an empty description in the run report and offer to run feature-creation's Backfill mode on them, passing the mapped property names (`fields.roadmap_description`, `fields.show_on_roadmap`) so Backfill reads and writes the same properties. In an attended run, you may start Backfill mode right after the validation results are shown; it drafts and shows every description, and nothing is written without the PM's OK. In an unattended run, only list and offer.
- **Not installed:** list the features, say that the feature-creation skill can fill them and that it exists (https://github.com/pendo-io/Product-Loop), and offer to install it. To install: in Claude Code, or anywhere you can write files and reach GitHub, offer to do it yourself, and on the user's OK download that skill's folder from the repo into `~/.claude/skills/feature-creation/`. Elsewhere (for example the Claude apps), tell the user to download the folder and upload it as a skill.

---

## Confidence, tiers, and the source map

The full decision procedure (field classification, what "positively corroborated" requires, the ≥95% bar, recency weighting, weakest-field gating, the audit format, and worked examples) lives in **`references/validation-logic.md`**. Load it on every real run; it is the heart of the skill.

The high-signal fields come from `config.high_signal_fields`. The defaults are the six below; at setup, map each to the tracker's actual property name in `config.fields`, add or drop fields to match the tracker, and set the per-field sources in `config.field_source_map`:

| Field (default name) | Primary corroboration | Secondary |
| --- | --- | --- |
| Current phase | Jira (or other issue tracker) status / workflow state | launch announcements in Slack, roadmap doc, date moved to current phase |
| Next phase | Jira roadmap / next status | spec doc, conviction doc, roadmap doc |
| Next phase estimated target date | related milestone target, Jira due date / fix version | roadmap doc |
| Uses AI? | spec doc, conviction doc, AI notes | feature record body |
| Regions available | launch / GTM doc, spec doc, conviction doc, region notes | Slack launch announcements |
| Early access details | early access doc, spec doc, conviction doc | Slack |

The **spec doc** is whatever design or spec document the feature record links to (`config.fields.spec_link`). The **conviction doc**, if the feature has one, is the pre-build decision record for it; treat both as external corroborating sources. Notes fields on the same record (AI notes, region notes) are PM-entered context: useful for understanding a value, but internal, so they don't count as *external* corroboration on their own.

Check each field's actual type in the tracker at setup (select, multi-select, date, text) and use its exact option labels. Don't assume a Yes/No field is a checkbox.

## Gotchas

Hard stops live in *Boundaries and authority* above.

- **Don't reconstruct the tracker's due logic.** If the tracker computes due flags, filter on them; some trackers (Notion formulas, for example) can only be filtered server-side, not read client-side. See `references/tracker-setup.md`.
- **Some tracker buttons can't be clicked via API.** If the tracker has a "Validate" button, write `Last Validated` = today directly instead.
- **Keep the two "due" rules apart.** A feature is routinely due when its validation cadence has run out, and pre-change due when its next phase is scheduled for today or tomorrow. Every run includes **both** kinds, and pre-change items go at the top because they can't wait.
- **Target-date rollups are read-only.** If the target date is a rollup from related milestones, corroborate against the milestones' own dates; don't write the rollup.
- **Scheduled runs can't open re-auth flows.** If a connector needs re-auth mid-run, catch the error, notify the PM ("Feature validation couldn't run, reconnect <tool>"), write `last_run.status = "failed_auth"`, and exit cleanly. Don't silently retry.
- **Coverage log must not grow unbounded.** Rotate per `references/sources.md`.

## Scheduling (optional)

Default cadence for scheduled runs is **daily** (`schedule.frequency = "daily"`), because validation cadences can be as tight as 7 days and pre-change items are due the day before; a weekly run would miss them. Silent-on-quiet-days keeps daily cheap. The skill can't register a scheduled task itself; `references/scheduling.md` has the steps. Without a schedule, the PM runs it on demand ("run feature validation").

## Files in this skill

| File | When to load |
| --- | --- |
| `SKILL.md` | Always |
| `config.example.json` | Shape of the saved config and the defaults |
| `state.example.json` | Shape of the run state (pending queue, coverage log) |
| `references/setup.md` | First run, or when the PM changes part of their config |
| `references/validation-logic.md` | Every real run: tier rubric, confidence rules, audit format |
| `references/tracker-setup.md` | Tracker mode: schema introspection, finding due features, write payloads |
| `references/sources.md` | When reading a source, logging coverage, notifying, or surfacing an insight |
| `references/scheduling.md` | Only when setting up or changing a schedule |

If this skill misbehaves, report it as an issue at https://github.com/pendo-io/Product-Loop; each fix lands as a named rule here.
