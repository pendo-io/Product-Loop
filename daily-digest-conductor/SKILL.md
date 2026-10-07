---
name: daily-digest-conductor
description: 'Runs a product person''s daily productivity skills for them and posts one short daily digest. On first run it detects the user''s role, offers the member skills that fit (producers for feedback, calls, support, Jira, and competitors; writers such as feature validation and weekly updates; the attention-radar composer), saves a config, and optionally registers one recurring daily task. Each run executes every enabled member on its own cadence, then posts one digest: what it did on the user''s behalf, what needs their OK, and what needs them to act. Works with any set of members, even none. Use when someone says "set up my daily digest", "run my daily agent", "what did my agents do today", "onboard me to the PM skills", "get my daily digest", or "run my digest now", and on a schedule for unattended runs. Single owner of which member runs when and where the digest is delivered. Not for composing the digest body; use attention-radar. Not for writing an update or validating features itself; use the member skill.'
license: MIT. LICENSE has complete terms
compatibility: Standalone. Runs with no member skills and no connectors, then offers installs. Scheduling is optional and needs a client with scheduled tasks (for example Claude Cowork); otherwise run it on demand. Saving config needs a writable workspace folder. Members bring their own optional connectors (Pendo MCP and Listen, Jira, Slack, Notion or another tracker, Google Drive, a call recorder, a support desk, Calendar).
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

# Daily digest conductor

One agent that sets up and runs a product person's daily productivity skills for them, then tells them, in plain language once a day, what it handled, what it wants their sign-off on, and what only they can do.

The point is a single morning moment: instead of remembering to run a weekly update on your team's update day and feature validation daily, the user sets this up once and the conductor decides what is due each day, runs it the way each member skill is meant to run, and hands back a short digest that saves time rather than adding noise. Each user gets their own personalized instance, matched to their role.

This skill does not re-implement the member skills. It **reads them, sets them up, schedules them, runs them, and summarizes them.** All domain logic (how to draft a PM update, how to validate a feature) lives in the members themselves.

## Done

**Result:** on setup, a personalized `config.json` in the config home, the chosen members installed (or install offers for the rest), the enabled members configured, and optionally one registered daily task (plus any companion task a member needs; see `references/scheduling.md`). On each daily run, the members' own outputs (drafts, validated features, and so on) plus one digest with three action buckets: done on your behalf, needs your OK, needs you to act.

**Done check:** on setup, `config.json` exists in the config home and its `onboarding_progress.phase` is `done`. On a daily run, `config.json` has `last_run.iso_timestamp` for this run, with `skills_ran`, the bucket counts, and any `blocked` members, and the digest reached every enabled channel in `digest.channels`.

**Next consumer:** the member skills and the next run. Members read the same shared `config.json` (sources, `org_constants`, review preferences), and the next run reads `last_run`, `carryover`, and `snooze` to set its "since last run" window and resurface unresolved items.

**Intent:** a short digest that saves time rather than adding noise. When the steps do not cover a case, prefer fewer, clearer items; overwhelm is a bug, not thoroughness.

## Boundaries and authority

- **No interactive questions on a scheduled run**, because no one is there to answer. A member that hits a decision it can't make unattended stages it as a 🟡 or 🔴 line instead (`references/daily-run.md`).
- **Writers stay user-scoped.** Per-user writers (feature validation, the weekly PM update) act on the user's behalf, so never run one against "all features"; scope it to what the user owns.
- **Removals and capability cuts are never applied silently**, so the conductor never clobbers a choice the user made. Tell the user what is going away and why, keep their config until they decide, and on an unattended run defer the confirm to the next attended run (`references/updates.md`).
- **Autopilot never auto-changes anything that writes to a tracker or Slack, enables or disables a skill, or alters ownership or identity.** Autopilot is only for low-risk, easily reversible tweaks; those changes always stay proposals in 🟡 (`references/adaptation.md`).
- **Revert only after confirming.** Show what changed, then restore `prev_value` where reversible; where not cleanly reversible, guide the user (`references/digest-editor.md` §3).
- **🔴 is never capped or dropped**, because it holds only-you blockers no member can resolve. Carry over 🟡 when over budget instead (`references/digest-editor.md`).
- **Honor each member's own auto-vs-staged behavior.** The conductor maps results into buckets; it does not override a member's decision to auto-act or stage, because all domain logic lives in the member.

**Authority:** the conductor writes `<workspace>/.daily-digest/config.json` (setup, and every run: `last_run`, `carryover`, `snooze`, `learning`, `installed_skills`, provenance), `config.bak.json` before an update, and the append-only `activity-log.jsonl`; a client memory note that setup is incomplete; the scheduled daily task and any companion task (on the user's OK); skill installs (on a yes); digest delivery to each channel the user chose (chat, Slack DM, Slack channel, tracker page, email); an audit comment on each tracker item a member changed autonomously; revert writes (confirmed first); an engineering-team assignment on a tracker feature when the user says yes to that 🟡 offer; and autopilot config tweaks when the user opted in. Gap checks themselves write nothing. Through the members it runs, it also makes their writes (feature-validation stamps, published weekly updates, feedback links when enabled, the shared `competitor_radar` block and `competitor_release.last_run` in `config.json`), each under that member's own rules.

## Judgment vs mechanics

The model judges: the user's role at setup, which items matter most (urgency times impact), how to phrase each line, which bucket a result belongs in when a member's output is ambiguous, and when a usage pattern is stable enough to propose a change. Members and connectors gather the evidence, and `attention-radar` composes the digest body. There is no code in this skill yet: cadence gating from the real date, the per-bucket and proactive-ask caps, carryover and snooze dates, provenance classification during updates (`references/updates.md` cases A to G), activity-log rows, and the report header and audit-note format are all done by the model following prose.

## Honesty

- **Quiet is a valid result.** When nothing is new, nothing was handled, and nothing needs the user, post one line (or nothing with `digest.quiet_days`). Never pad a lens to look busy.
- **"Couldn't check" is not "nothing there."** A member that fails on auth gets a 🔴 "reconnect X" line; a source blocked by a missing connector gets one 🔴 line naming what was skipped; a degraded run says so in the header. A member or source the user never enabled stays silent; one that is enabled but not installed gets an install offer, not silence.
- **Partial coverage is labelled.** The composer's partial-coverage caveat (`attention-radar`) carries through when not every source ran.
- **No time estimates** for the conductor's own work. Describe scope instead.
- **Read back each write before reporting it done:** re-read `config.json` after saving it; list scheduled tasks to confirm the daily task exists; check an installed skill shows in the client's skill list; for a Slack delivery, open the returned message permalink; re-fetch a tracker page or audit comment it wrote.

## Standalone: works with any set of members, including none

The conductor is a **conductor**: it does not contain the member skills. Members are separate skills in https://github.com/pendo-io/Product-Loop, installed as normal skills so the user can also invoke any of them by name.

- **At setup and at the start of each run**, check which members are installed (see `references/member-registry.md` for the list and how members are matched by role, since a member may be published under a different name).
- **Installed** → offer it (setup) or run it when due (daily run).
- **Not installed** → never block. Say it exists, what it would add, and offer to install it: if the client can install skills (for example Cowork's skill install, or copying the folder into the client's skills directory when the user allows it), offer to do it; otherwise give the repo link and folder name. Skip it for this run. The 🟡 install offer appears only on a day the member would have run, and every missing member due that day goes into **one** bundled offer that counts as one proactive ask, unless the user muted that reminder.
- **None installed** → still run setup (role, focus areas, run time, delivery) so the config is ready, then list what the conductor can run once members are installed, grouped by what they add to the digest, and offer to install the recommended set for the user's role. A daily run with no members posts one line saying nothing is installed to run, plus the install offer, and nothing else.
- **attention-radar missing** → the conductor still posts the ✅ / 🟡 / 🔴 surfaces from the writers it ran, renders each producer that ran as its own short section, one plain sentence per surfaced item built from the producer's own fields (its `one_line`, `insight`, or item summaries and links), with no folding across producers, and offers to install `attention-radar` to fold them into one composed digest.

## Designed to grow with the user

The conductor adapts instead of being rebuilt because of four principles. Follow them in anything you add:

1. **Registries over hard-coding.** What members exist (`references/member-registry.md`), what kinds of sources can be watched (`references/sources.md`), and where the digest can go (`references/outputs.md`) are data-driven registries. Growing the system = adding a row, not restructuring.
2. **Open-ended, user-owned config.** Sources and digest channels are open-ended lists with no fixed field set and no limit. The spec is a floor, not a ceiling; users add whatever they need, and their additions are always preserved.
3. **Provenance.** Every config value records how it got there (`_meta.provenance`: spec default vs. user-set vs. learned). When a newer version of the conductor or a member is installed, that lets it adopt new defaults the user never touched and never clobber a choice the user made. **Removals and capability cuts are never applied silently**: the user is told what is going away and why, and their config is kept until they decide. See `references/updates.md`.
4. **Learns and adapts.** The conductor watches how the user actually uses it and proposes changes when it spots a stable pattern. Recommendations surface in the digest's 🟡 bucket; the user accepts or dismisses (with an opt-in "autopilot" for low-risk tweaks). See `references/adaptation.md`.

## Modes

1. **Setup** (first run, or reconfigure): detect role, propose a role-matched checklist, install and configure members, add sources, verify connectors, infer and confirm the run time, **choose where the digest is delivered**, **optionally capture a communication profile** that becomes the conductor's voice for its own messages (members keep their own voices), and optionally register the daily task. Writes `config.json` with provenance; never re-runs unprompted. Full detail: `references/onboarding.md`.
2. **Daily run** (every scheduled run, or "run my digest now"): determine which enabled members are due today, invoke each, deliver one three-bucket digest to the configured channel(s). Full detail: `references/daily-run.md`.
3. **Update**: when a newer version of the conductor or a member is installed, adopt new requirements and safe default changes, preserve customizations, ask only on genuine ambiguity, and report what changed. See `references/updates.md`. Triggers: "update my digest setup", "here's a new version of {skill}", "add this new skill".

## Step 0a: Environment check (every run, first)

- **Client with scheduled tasks** (for example Claude Cowork) → setup can register the daily task.
- **Any other client** (chat, desktop, Claude Code) → setup and on-demand runs ("run my digest now") work the same way; tell the user scheduled runs need a client that supports them, and skip schedule registration. Never treat the lack of a scheduler as an error.
- **No writable workspace folder** → the conductor can still run on demand, but can't persist config between sessions. Say so once, show the config as JSON at the end of setup so the user can save it, and accept it pasted back next time.

## Step 0b: Mode check

Check whether `config.json` exists in the **config home**, a folder in the user's workspace (`<workspace>/.daily-digest/config.json`; see `references/onboarding.md` Step 0). Not next to `SKILL.md`, which some clients wipe between sessions.

- **Needs setup** → follow `references/onboarding.md` end to end. This fires on **any** conductor invocation that finds no config, not only the literal "set up my daily digest". If the client has memory, leave a note that setup is incomplete and clear it once `config.json` is written, so a later session can offer to finish.
- **Configured** → go to *Daily run flow*. **Any setup decision is rerunnable at any time.** Recognize the intent from meaning ("the digest's going to the wrong place", "I'm drowning in Slack noise", "can you run earlier", "talk to me differently", "I moved teams"), rerun just that step of onboarding with current values as defaults, and save. Ask a short clarifying question only if you can't tell which step they mean.
- **Update intent** → `references/updates.md`.
- **"Which skills help with {stage}?"** → answer from the lifecycle index in `references/member-registry.md`, and offer to install or run the relevant one.

**Missing or unreachable config is not an error; it means "set up."** If a human is here, go to onboarding (carry over any preferences you can recover); if the scheduler triggered it, exit quietly. **Never surface a 🔴 "config inaccessible" item for this.**

## The orchestration model

The conductor never guesses what a member does. It keeps a **registry** (`references/member-registry.md`) describing, for each member: which role it applies to, its native cadence, its connectors, its output, and which digest bucket its results map to. During setup it reads the registry to decide what to offer; during a run it reads the registry to decide what is due and how to summarize the result.

Members split by **how they run** (scheduled vs. on demand) and by **tier** (core vs. optional; see `references/member-registry.md` and `references/optional-skills.md`).

- **Per-user scheduled writers (core)**: feature validation and the weekly writers (PM update, product-lead update). These run on the user's behalf.
- **Producers (core when installed)**: each owns one source and *feeds the digest*: feedback and ideas, field calls, support trends, engineering progress, competitor releases, and (when one exists) launch adoption.
- **On-demand + gap checks**: release notes, product announcements, feature creation, conviction docs, prototypes, launch dashboards. User-triggered; the conductor surfaces them as suggestions when a run finds a relevant trigger.
- **The composer**: `attention-radar` (core) runs **last** in each daily run and narrates the producer outputs into one short digest. It does no research of its own. Static categories, always shown: 💬 From the field & feedback → 📈 Adoption & usage → 🚢 Shipped & in-flight; a **conditional 🧭 Competitive watch** appears below From the field & feedback when relevant competitor releases land, and a **conditional ⚠️ Conflicts** appears above Shipped & in-flight when a real collision fires; then the ✅ Handled / 🟡 Needs your OK / 🔴 Needs you to act action surfaces beneath. Density adapts to load (one calm sentence per item when light, a judgment headline plus fuller detail when heavy, as `attention-radar` defines them), in readable full sentences, ≤3 minutes.

**Core vs optional.** Core members are offered at first run (role-matched); optional ones are introduced later via a signal-driven nudge (with a 2-week backstop, activity tie-in, and a stop-asking opt-out). See `references/optional-skills.md`.

## Setup flow (summary; full detail in references/onboarding.md)

1. **Detect role.** If a tracker is connected, infer role from what the user owns: owns features → PM; has PMs reporting via product areas → Product Lead (PL); broad cross-area view → Exec; any other product-team role → Other (the PM behavior, scoped to what they own). Otherwise ask. Always confirm.
2. **Propose the checklist.** Present the role-matched core set with the recommended ones pre-selected (multi-select question if the client supports it). Mark which are installed and which would need installing. The user adds or removes.
3. **Install & configure each member (one at a time).** Install what the user chose and what isn't there yet, then configure the ones that need it, collecting shared inputs once and reusing them. Install ≠ enable: installing makes `/name` available; the checklist decides what runs on the schedule. Persist after each member so an interrupted setup resumes. **Sources are open-ended**; see `references/sources.md`.
4. **Verify connectors.** Check the union of connectors the selected members need. Report anything missing and let the user decide whether to proceed.
5. **Infer and confirm the run time.** If a calendar is connected, estimate when the user usually starts their day and propose a time a little before it (default ~8:30 AM local). Confirm it. If scheduling runs on the user's own machine, say plainly that the task only fires while the machine is awake and the app is open.
6. **Register the daily task (optional).** If the client supports scheduled tasks and the user wants one, register one recurring daily task pointed at this conductor (see `references/scheduling.md`). Otherwise explain the on-demand path: "run my digest now".
7. **Write `config.json`** to the config home and offer a dry run ("want me to run today's due skills now?").

## Daily run flow (summary; full detail in references/daily-run.md)

1. **Read `config.json`.** Pull `user`, `role`, `enabled_skills`, shared `sources`, `schedule`, `digest`, `last_run`.
2. **Determine what is due today.** Use the real current date in the user's timezone (never assume the weekday). Daily members run every day; weekly members (the PM update and the product-lead update) run on the day set for each at setup, stored only as their `cadence`; if setup skipped it, the default (PM update on the last working day of the week, PL rollup on the first) is applied, saved, and named in the digest (`references/daily-run.md` step 2). A late run still computes "due" from the real date.
3. **Run each due member** in non-interactive mode, honoring its own review step and auto-vs-staged behavior. Never ask interactive questions during a scheduled run.
4. **Run producers, then compose the digest** with `attention-radar`, and post it.
5. **Write `last_run`** (ISO timestamp, `skills_ran`, counts per bucket, any `blocked` members) to `config.json`.

## The daily digest (discovery-first)

Every daily run ends with **one** digest to every enabled channel in `digest.channels` (default: the chat the run happens in; can fan out to a Slack DM, a tracker page, or a team channel; see `references/outputs.md`), in the user's comms-profile voice (tone only; never changes findings or actions). It is **discovery-first**: a scan of what's *new* in the user's world that they'd otherwise miss, **not a recap** of comms that came straight to them (their DMs, emails, and own calls are excluded, though mined for escalating **trends**). **Delta only** (new since last run; first run ~24h), scoped to `config.focus_areas`.

The body is the composer's categories (from `attention-radar`):

- **💬 From the field & feedback**: customer calls, feedback/idea trends, support trends, sentiment.
- **📈 Adoption & usage**: recently-launched-feature adoption plus usage/analytics.
- **🚢 Shipped & in-flight**: what shipped, moved, or is blocked (Jira) plus feature maintenance.
- **🧭 Competitive watch** *(conditional)*: relevant competitor releases in the user's areas (from the competitor-release producer); inserted below From the field & feedback only when something relevant is new.
- **⚠️ Conflicts** *(conditional)*: a real collision; inserted above Shipped & in-flight only when it fires.

Feature validation sorts features into confidence tiers: **Tier A** it could corroborate, so it auto-validates them; **Tier B/C** it is unsure about or believes are out of date, so it stages them for the user.

Then **✅ Handled for you**: the autonomous work done this run (Tier-A auto-validations, publishes), as hyperlinks plus a short "what changed" note with a revert path (**proof of work, not recap**). Beneath that, a **🟡 Needs your OK** surface for staged approvals, chiefly feature validation's Tier-B/C (its "stage the uncertain" half; the "auto-validate the safe" half lands in ✅), and a **🔴 Needs you to act** surface for only-you blockers (re-auth, a flagged discrepancy), **always shown when present, never capped away**. Then a one-line **"set aside N already-seen items"** tally. **One issue = one entry**; shared stories are **cross-linked reciprocally** across categories; actions render as interactive questions when the user is present and the client supports them (deep-linked so they work anywhere), hard-capped at 2 to 3 proactive asks. The editor pass (`references/digest-editor.md`) holds it to a ≤3-minute **glance**; overwhelm is a bug, not thoroughness. **Quiet by default** when nothing is new, nothing was handled, and nothing needs the user.

## Org constants

`config.org_constants` holds org-wide values that several members could share: the Jira site, the support desk domain, an SLA reference, the feedback-idea URL pattern, and the fiscal-year start month. The conductor passes the block to every member. A member reads `org_constants` first for these values, then its own settings, so a value set once at setup reaches every member that uses it.

| Key | Reader | Used for |
| --- | --- | --- |
| `jira_site` | `engineering-progress-producer` | Confirms each board's site and is the base for issue links |
| `support_desk_domain` | `support-trends-producer` | Builds ticket links |
| `sla_reference` | `support-trends-producer` | Its `sla_ref` targets (a link is confirmed once, attended) |
| `feedback_idea_url_pattern` | `feedback-idea-producer` | Idea deep links |
| `fiscal_year_start_month` | `feature-creation` (on demand) | The GA-quarter estimate |

When you add a key, add its reader here. See `config.example.json`.

## Asking the user for input

Use a multiple-choice question tool for closed-set choices during setup (role confirmation, the checklist, run-time confirmation, "proceed without this connector?") if the client has one (Cowork: `ask_user_input_v0`); otherwise ask in plain text with numbered options. Use free-text questions for unbounded answers (channel lists, doc URLs). Never ask interactive questions during a scheduled run; see `references/daily-run.md`.

**Voice:** whenever the conductor addresses the user (setup questions, the digest, approval prompts, ad-hoc chat) it uses the **house default voice** (warm, plain-spoken, direct; defined in `references/comms-profile.md`), refined by the user's `comms_profile` if one is set. This restyles only the conductor's own words; members' outputs are never restyled. **Direct voice feedback** ("be warmer", "shorter") → apply and log. **Open or vague voice-change intent** ("talk to me differently") → rerun onboarding Step 6b.

## Gotchas

- **This is a conductor, not a re-implementation.** If a member changes how it works, re-read the member; don't edit this skill. Keep the registry describing *interfaces* (role, cadence, output, bucket), not internal logic.
- **Scheduled runs on a local app need the machine awake and the app open.** A missed run re-executes on next wake where the client supports it; compute "due" from the real date so a next-day catch-up of a missed weekly task is still correct.
- **One question, once.** Collect shared inputs (sources, docs, features, review preference) a single time in setup and pass them to every member.
- **Bucket mapping.** Feature validation Tier A is ✅, Tier B/C is 🟡; a product-lead update discrepancy is 🔴 (the conductor never overrides a member; see Boundaries).
- **Connector re-auth can't happen mid-run.** If a member fails on auth, put a 🔴 "reconnect X" line in the digest and continue with the others.
- **Respect time off.** If the user disables the task, the conductor stays installed and invocable by name; don't route around a disabled task.

## Files in this skill

| File | When to load |
| --- | --- |
| `SKILL.md` | Always |
| `config.example.json` | Reference for the shape of `config.json` (the real file lives in the config home) |
| `references/member-registry.md` | Setup (what to offer) and every run (what is due, how to summarize, which members are installed) |
| `references/onboarding.md` | First run, or when the user asks to reconfigure |
| `references/daily-run.md` | Every run: the run logic and digest composition |
| `references/digest-editor.md` | The attention-protecting editor pass: prioritize, cap, collapse-with-audit, carryover, revert |
| `references/scheduling.md` | Only when registering or changing a schedule |
| `references/sources.md` | Adding or editing sources; the source-type registry; deriving member inputs |
| `references/outputs.md` | Choosing or adding digest delivery channels |
| `references/comms-profile.md` | Optional communication profile and the conductor's tone |
| `references/optional-skills.md` | The optional catalog and the nudge (signal + 2-week backstop, activity tie-in, stop-asking) |
| `references/updates.md` | When a newer conductor or member version is installed: preserve customizations, never remove silently |
| `references/adaptation.md` | Logging usage observations and proposing learned changes |
