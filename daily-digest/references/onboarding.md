# Onboarding: guided setup

Run this the first time the conductor is invoked with no config, or when the user asks to reconfigure (add/remove a member, change run time, role change). Onboarding also **installs the members the user picks** (where the client can install skills) and saves all settings to `config.json` in the config home (Step 0). When only part changes, rerun just that section and save in place.

Goal: the user answers each thing **once**, ends up with the right members configured, and, if their client supports it, has one daily task registered. Keep it conversational; use a multiple-choice question for closed choices (if the client has one), prose for free text.

## Step 0: Establish the config home (do this first)

`config.json` must live somewhere that survives across sessions **and** that each member can read when it's invoked on its own. Use the user's workspace folder, not the folder next to `SKILL.md` (some clients wipe the skill area between sessions).

1. Confirm a workspace folder is available. If none is, say config won't persist between sessions, carry on, and at the end show the config JSON for the user to save (they can paste it back next time).
2. Use `<workspace>/.daily-digest/config.json` as the config home; create `.daily-digest/` if absent.
3. If the client has memory, **write a breadcrumb** naming the folder, so a later session that can't see the config can name the folder to reconnect instead of re-onboarding blindly.
4. Record the folder in `config.onboarding_progress.config_home` and set `onboarding_progress.phase = shared_block`.

## Step 1: Detect and confirm role

1. If a tracker with feature ownership is connected, infer role from what the user owns:
   - Owns features and no PMs report to them → **PM**.
   - Product areas roll up PMs to them → **Product Lead (PL)**.
   - Broad view across many product areas / exec → **Exec**.
   - Any other product-team role → **Other** (the PM behavior, scoped to what they own).
2. With no tracker, ask directly.
3. **Confirm, don't assume.** "I think you're a **{role}**; is that right?" with the other roles as options.

## Step 2: Propose the member checklist

Read `references/member-registry.md` → "Tiers" and "Role → default-enabled set". Present **only the core set** for their role, recommended items **pre-selected**, as a multi-select. Mark each as installed or "needs installing". Don't put optional members in this checklist; mention once that "there are a few more optional skills I'll offer down the line." Give a one-line "what it does + when it runs" per item. Example for a PM:

- ☑ Daily feature validation: checks your feature records are accurate (runs daily)
- ☑ Weekly PM update: drafts your weekly update (runs on your team's update day)
- ☑ Producers: feedback and ideas, field calls, support trends, engineering progress, competitor releases (run daily, feed the digest)
- ☑ Digest composer: composes the digest (runs last, every day)

Let the user tick or untick. Record the result as `enabled_skills`.

**If nothing is installed**, still record the user's picks (the recommended core set if they just accept) in `enabled_skills` with `status: not_installed`, so each starts running as soon as it's installed. Finish setup anyway (Steps 3, 6, 6a, 6b, 8), then say what the conductor would run once members are installed and offer to install the recommended set.

**Enable ≠ install.** Ticking decides only what runs on the daily schedule. Installing makes a member invocable by name.

Introduce the **daily digest** here: `digest-composer` leads every digest by **surfacing what's new and worth attention** in the user's world (customer signal first, then conflicts, then what's on track), **excluding the comms that already came straight to them**. Set expectations plainly: *it surfaces what you'd otherwise miss; it is **not** a to-do list or reminder system, so keep your own for that.* The more sources and producers are connected (Step 3), the sharper it gets.

## Step 3: Collect shared inputs once

Several members need the same inputs. Gather them once and reuse.

**Sources (open-ended; add as many as the user wants).** Follow the add loop in `references/sources.md`: repeatedly ask "what else should I watch?" (free text), infer the `type`, resolve the `ref` via the right connector, and append to `config.sources` until the user is done. Start with their key Slack channels and tracker pages if those are connected; more can be added any time with "add a source".

**Other shared inputs** (stored under `preferences` and top-level keys, not `sources`):

- **Name and work email**: store as `config.user.name` and `config.user.email`. Members use them to recognize the user in their sources, for example competitor-release-producer's owner match by name or email. If a connected tracker or the client already knows them, confirm instead of asking. If the user skips the email, members match by name only.
- **Owned features**: derive from the tracker's owner field if connected and confirm ("I found N features assigned to you; keep all?"). Otherwise ask, or accept a pasted list.
- **Current focus areas / features**: ask *"What product areas or features are you focused on right now?"* Store as `config.focus_areas`. This **anchors the digest's discovery scan**: calls, Jira, feedback, and tracker activity are filtered to these. If they skip it, fall back to owned features and say so.
- **Weekly update day** for the weekly writers: ask which day the user's team files weekly updates (and, for a PL, the day the rollup is due, after the PM updates it reads). Store it as each weekly writer's `cadence` (`weekly-<day>`); under the conductor this is the only place the day lives, and the writer's own `schedule` section stays unused. **If the user skips it, use the default and say so:** the PM update on the last working day of their week, the PL rollup on the first working day of their week, saved with provenance `set_by: spec_default`.
- **Review-step preference** for the weekly writers: publish-and-notify-for-review vs publish silently.
- **Org constants** (ask only for what the enabled members need; skip any the user doesn't have): Jira site, support desk domain, SLA reference (a link or the targets pasted in), the feedback-idea URL pattern for deep links, fiscal-year start month (calendar year if they have none). Store under `config.org_constants`; each key's reader is listed under Org constants in `SKILL.md`.

Don't store per-member source arrays as separate keys; the conductor derives those per member from the generic `sources` list at run time.

**For the digest, prompt for its signal sources** while adding sources. Offer these as a menu and always close with *"anything else you want it to watch?"* (nothing is hardwired):

- **Pendo (usage + Listen feedback)**: scope to the user's product area. Resolve and store `config.pendo.subscription_id`, `app_id`, and `product_area_ids` for their area so usage and feedback reads are *their-area* signal. Add `pendo_usage` + `pendo_listen` sources. Recommend it when feedback-idea, launch-watch, or conviction-doc are enabled; never require it.
- **Call recorder** (for example Gong): their team or library ref; add a `call_recordings` source (read by `field-qual-producer`).
- **Docs** (conviction docs, strategy), key **Slack** channels, **email** (optional).
- **Call notes** resolve through the fallback chain in `references/sources.md`: the call recorder first (if it fails to connect, prompt the user to re-authorize); then meeting notes in the tracker or docs; then any other notes tool the user names. Store the resolved provider in `config.radar.call_notes_source`.

## Step 4: Install & configure each member (one at a time)

Install and configure **one member at a time, persisting after each**, so an interruption always leaves a valid config. Set `onboarding_progress.phase = installing`; resume from `onboarding_progress.last_skill`, cross-checked against the installed skill list.

For each chosen member:

1. **Install it if needed.** If the client can install skills, install it from https://github.com/pendo-io/Product-Loop (or the user's own copy). Otherwise give the folder name and repo link and move on; the run will offer it again until it's installed or muted. Record `installed_skills.<name>` = `{ installed, version, mute_reminder: false }`.
2. **Configure it (delta only).** Most on-demand members need nothing beyond the Step-3 shared inputs; just confirm. For a member with its own onboarding (the weekly writers, feature validation, the producers), drive it now **passing the Step-3 shared inputs** so the user isn't re-asked, and let it own its member-specific questions.
3. **Persist immediately.** Write `installed_skills.<name>`, any per-member config, and advance `onboarding_progress.last_skill`.

**Don't register each member's own scheduled task.** The conductor drives everything from its single daily task (Step 7). If a member's onboarding wants to self-schedule, skip that and tell the user. `launch-watch` is no exception: its dashboard's Generate button runs in the page and needs no task, re-pinning for everyone is a `launch-watch` run (by hand, or the conductor's `weekly-<day>` cadence, `references/optional-skills.md`), and it never registers a task of its own here. A user can still ask for an extra standalone schedule for one member later; that member still reports in the main digest unless they opt it out.

## Step 5: Verify connectors

Compute the connector union for the enabled set (registry "Connector union"). Check each. Report in one line, for example:

> Connectors: Slack ✓ · tracker ✓ · Jira ✗ (engineering-progress-producer can use it or another issue tracker) · Pendo ✓ · Calendar ✓

If something is missing, ask whether to proceed anyway (the member needing it is skipped with a 🔴 line until connected) or pause to connect it. Don't hard-block. If Pendo is missing and a Pendo-backed member is enabled, offer to walk the user through connecting it now rather than letting the first run fail silently.

## Step 6: Infer and confirm the daily run time

1. **Infer from the calendar** if connected: look at the last ~2 weeks to estimate when the day usually starts (the earliest recurring meeting or first weekday event) and propose a time a little before it (a first meeting at 10:00 → propose 9:30). With no calendar, the default is **8:30 AM** in the user's timezone.
2. **Confirm the time**, offering a couple of nearby alternatives and an "other" option.
3. If the scheduler runs on the user's own machine (for example Cowork), **state the caveat plainly**: the task only fires while the machine is awake and the app is open; a missed run re-executes when the machine wakes.

Store under `schedule` (`time`, `timezone`, `cadence: "daily"`). If the user won't be scheduling, still store the time; it sets the "since last run" window for on-demand runs.

## Step 6a: Name the report, and choose where it goes

**Name the daily report.** Offer, in order: **Daily digest** (default), **Morning digest**, or **their own name**. Store as `config.digest.report_name`. A `report_name` already in a user's config is their choice: keep it, including a name from an earlier version's list. It's the headline label and how the conductor refers to the report.

**Then choose where it goes**, from the output-channel registry (`references/outputs.md`): **this chat (default)**, Slack DM, a Slack channel, or a tracker page; pick one or several. Default to chat if they skip. Resolve each `ref`. Store in `digest.channels` and mention it's changeable any time ("change where my digest goes").

## Step 6b: How should the conductor talk to you?

Shapes the conductor's voice for every message it sends the user. It never restyles what members write. Full detail: `references/comms-profile.md`. This step is **optional**.

1. **Ask for a communication profile first.** *"Do you have a personality or communication-style profile (for example an Insights Discovery report)? If so, share it and I'll match how it says you like to be communicated with."* Accept pasted text, a link, or a file (a PDF export is common). If provided: copy **verbatim** its "how to communicate with me" do's and don'ts, plus a short tone cue, into `comms_profile.insights`, then build the `voice_directive` **primarily from the profile**.
2. **If no profile: house default plus a quick tune-up (optional).** *"I'll talk to you in a warm, plain-spoken way by default; in a sentence or two, anything you'd change?"* Capture it into `voice_directive` as a refinement of the house default.
3. **Region/locale: ask, don't assume.** *"Want me to flavor my phrasing by region, for example British English spelling and idiom?"* Only on yes, capture `comms_profile.region` with **low weight**.
4. **Intensity**: **light** / **balanced** / **full**. Store as `comms_profile.intensity`. Only worth asking if a profile or preference was captured.

If they skip everything, write **no** `comms_profile`; the house default is used. Voice changes later: **direct feedback** ("be warmer", "shorter") → apply and log; **open or vague intent** ("talk to me differently") → rerun this step.

## Step 7: Register the schedule (optional)

Follow `references/scheduling.md`. If the client supports scheduled tasks and the user wants one, register one recurring daily task at the confirmed time. If an enabled member needs a companion task, offer it too. Confirm and tell the user where to see or edit it. If there's no scheduler, explain the on-demand path ("run my digest now") and move on.

## Step 8: Write config and offer a dry run

Write `config.json` to the config home (shape in `config.example.json`) with `user`, `role`, `enabled_skills`, `installed_skills`, `sources`, `focus_areas`, `org_constants`, `schedule`, `digest.channels`, `comms_profile` (if provided), `onboarding_progress.phase = done`, an empty `last_run`, and `_meta`: set `conductor_version` and a `provenance` entry for each value written (`set_by: spec_default` for defaults, `set_by: user` for choices, with `base_value` = the default). Clear any "setup incomplete" memory note. Then offer: "Want me to run today's due skills now as a test?" If yes, run the daily run flow in interactive mode and show the three-bucket digest.

## Reconfigure entry points

**Every setup step is rerunnable at any time; recognize the intent, don't wait for exact wording.** Infer which decision they want to revisit, confirm briefly, rerun that step (current values as defaults), save, and update provenance. Only rerun the whole flow if they clearly want a full redo. When unsure which step they mean, ask a short clarifying question.

- "Add/remove a source" / "watch this channel too" / "stop watching X" → the add/edit/remove flow in `references/sources.md`.
- "Connect my call recorder / Pendo for my area" / "the digest is too tracker-heavy" → rerun the Step 3 signal-source prompts.
- "Change my focus areas" / "I'm working on X now" → update `config.focus_areas`.
- "Change where my digest goes" → Step 6a.
- "Rename my report" → update `digest.report_name`.
- Voice changes → direct feedback = apply + log; open/vague = Step 6b.
- "Update my digest setup" / "here's a new version of {skill}" → `references/updates.md`.
- "Tune it for me" / "you decide" / "stop suggesting X" → `learning.autopilot` or `learning.dismissed_patterns` per `references/adaptation.md`.
- "Add/remove a skill" → Steps 2 to 4 for the delta only.
- "Add a producer / optional skill" → that member's setup slice per `references/optional-skills.md`.
- "Stop offering optional skills" → `optional_skills.stop_asking = true`.
- "Change my run time" → Steps 6 to 7.
- "I'm a PL now" / role change → Steps 1 to 2, then reconcile enabled members.
- "Redo my setup" → full flow, existing answers as defaults.
