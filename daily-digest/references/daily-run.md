# Daily run flow

Load this on every scheduled run and on an on-demand "run my digest now". The run is **non-interactive** when fired by a scheduler; an on-demand run with a human present may ask questions.

## 1. Load state

Read `config.json`: `user`, `role`, `enabled_skills`, `installed_skills`, `sources`, `preferences`, `org_constants`, `schedule`, `digest`, `comms_profile`, `last_run`. If `config.json` is missing, this instance was never set up: if a human triggered it, start onboarding; if the scheduler triggered it, exit quietly. A missing config is a setup situation, **never** a 🔴 "config inaccessible" digest item.

**Renamed skills.** If the config still names a skill by its pre-1.1.0 name (see the map in `references/updates.md` → "Renamed skills"), run that migration first. It backs up `config.json` before changing anything and is safe on an unattended run. The scheduled-task handover waits for an attended run: on every attended run where `schedule.task_id` isn't `daily-digest-daily` and the scheduler lists a task under the old id, offer it as one 🟡 (`references/scheduling.md` → "Handing over from the old task id"), unless `schedule.task_handover_declined` is true.

`sources` is an open-ended typed list. Ignore any source with `enabled: false` or an unknown `type` (flag unknown types as 🔴). Connector-gate the rest per `references/sources.md`: a source whose connector isn't connected is skipped with one 🔴 line, not a run failure.

**Check which members are installed** (`references/member-registry.md` → "Finding members"). An enabled member that isn't installed is skipped. It gets a 🟡 install offer only on a day it would be due (step 2), and all of today's missing members share **one** bundled offer that counts as one proactive ask (unless muted). If **no** member is installed, post one line (*"Nothing is installed for me to run yet"*), list what would run for the user's role, offer to install it, write `last_run` (step 5), and stop.

## 2. Determine what is due

Get the **real** current date and weekday in the user's timezone; never assume the weekday (a skipped run may execute a day late). For each enabled member, read its cadence from the registry and apply the gating rule:

- daily (feature validation, producers) → due;
- weekly-<day> (the PM update and the product-lead update, each on the day set at setup) → due if today is that day. **If the day was never set** (setup skipped the question, so `cadence` is still the placeholder `weekly-<day>` or empty), apply the default first: the PM update on the last working day of the user's week, the product-lead rollup on the first working day of their week. Save it to `cadence` with provenance `set_by: spec_default`, and say once in the digest which day it used and how to change it;
- `launch-watch` with its weekly refresh enabled (cadence `weekly-<day>`) → due on that day; it re-pins the dashboard (see the ✅ section below);
- a weekly producer or member → due on its assigned day (weekly members are spread across the week so no single day gets heavy); optional members run only if the user enabled them;
- a member with a companion task (see `references/scheduling.md`) runs that part from the companion task, not here.

Only **core** members (and **enabled optional** members) run. Never auto-run an optional member the user hasn't enabled. Also skip any member that isn't installed or is blocked by a missing connector (record these for the 🟡 / 🔴 surfaces).

If a weekly member's window was missed entirely (the machine was off on the update day and this is the next run), still run it with the correct back-dated window and note in the digest that it's catching up.

## 3. Run each due member (non-interactive)

Invoke each due member in scheduled / non-interactive mode. Pass each weekly writer its day from its `cadence` (the single source of truth for the day under the conductor; the member's own `schedule` section is not used here). Build its inputs by **deriving** them from the generic `sources` list (filter to sources whose `consumers` include this member, or `"all"`, and whose `type` it can consume; shape them into the arrays it expects) per `references/sources.md`. Pass the per-user writers the user's identity in the tracker so their ownership filter is set, and pass `org_constants`. Honor each member's own behavior:

- **Never ask interactive questions on a scheduled run.** If a member hits missing config or a decision it can't make unattended, it should stage the item, not block. Capture that as a 🟡 or 🔴 line.
- **Respect the member's auto-vs-staged logic.** Feature validation decides Tier A (auto) vs B/C (staged); the weekly writers honor their `review_step`.
- **Catch connector/auth errors per member.** One member failing on auth must not abort the others. Record a 🔴 "reconnect X" line and continue.
- **Exec roll-up:** instead of writing personal updates, read the latest PL and PM updates across the user's product areas and compose a summary for the digest body itself.

Collect, per member, a short structured result: what it did, links, and any staged or blocked items.

### Gap checks: run every day regardless of cadence

After the due members, run the lightweight gap checks from the registry against the user's owned work. Core one, only when a tracker with the needed fields is connected:

- **Unassigned engineering-team check**: owned features whose engineering team and/or engineering owner field is empty → a 🟡 line offering to assign one (suggest a team from the feature's product area if the tracker records one). If several features lack a team, offer a **single bulk assign** rather than one nag per feature.

These checks never write anything; they only produce suggestions. Skip a gap once resolved; don't re-nag day after day.

## 3c. Producers, then the composer: the body of the digest

Run the due **producers** first and collect each one's structured output, then run the **composer**, resolved in this order: the skill named in `config.members.composer`, if it is installed; else an installed `digest-composer`; else an installed composer under its pre-1.1.0 name that is still the full composer, not the deprecated stub (a stub's description starts with "Deprecated"). It narrates those outputs into the digest and does **no** research of its own. **Producer ordering:** run `support-trends-producer` **before** `engineering-progress-producer`, so engineering-progress can consume `support_trends.eng_wait_queue` in the same run and hand the composer one merged 🔴 per escalation. **Definition:** `support_trends.eng_wait_queue` = support-trends-producer's `eng_waiting` entries with `queued_for_jira: true` and no `linked_jira` (escalations waiting on engineering with no linked bug; `linked_jira: null` means no linked bug, rendered as "no linked bug"). engineering-progress-producer's `decisions` key (product-consequential decisions: a scope cut, a descoped epic, a bug or story closed Won't Do (or not planned, or canceled), or a committed item moved to a later release) is passed to the composer like its other keys and lands in 🚢 Shipped & in-flight. `competitor-release-producer` has no cross-producer dependency; run it anywhere in the producer set, and pass it the shared `competitor_radar` block from `config.json` (competitors, sources, fields, `releases_db`, `competitors_watch`, `delta_anchor`, seen list, saved triage answers) so the user sets it once; it writes `seen` and `triage_answers` back to that block, and its own delta anchor to `competitor_release.last_run` (never to the conductor's top-level `last_run`, which is this run's record). **Pass `config.role` to the composer** so it picks the altitude: every lens in full for `PM` / `Other`; for `PL`, the cross-cutting leader synthesis (producers as corroboration, no per-area dumps); `Exec` → the roll-up in step 3, not the composer's per-user branch. The behaviors the daily run depends on:

- **Delta only**: cover only what's new since `config.last_run` (first run ~24h). Never a history dump.
- **Static categories**: 💬 From the field & feedback → 📈 Adoption & usage → 🚢 Shipped & in-flight (always shown; a calm line when quiet), plus a **conditional 🧭 Competitive watch** below From the field & feedback and a **conditional ⚠️ Conflicts** above Shipped & in-flight, each shown only when it fires. Density adapts, matching `digest-composer`: light (fewer than 8 surfaced items and no dominant story) = one sentence per item, no headline; heavy (8 or more items, or one dominant story such as a 🔴 item touching several lenses) = a judgment headline plus fuller detail.
- **Exclude already-seen**: drop anything that came directly to the user (their DMs, emails, own calls) or that they've handled, but **mine that pile for escalating trends** and surface the *trend* (not the items) as a heads-up. Tally "set aside N."
- **Customer signal first, the record last**: the category order puts 💬 first and 🚢 last; scoped to `config.focus_areas`.
- **One issue = one entry**, and **cross-reference** shared stories across lenses. Reads whatever's connected, names what it skipped, never blocks.

**If no composer resolves** (the deprecated stub doesn't count), skip composition: render each producer that ran as its own short section, one plain sentence per surfaced item built from the producer's own fields (`one_line`, `insight`, or item summaries and links), with no cross-producer folding; post the ✅ / 🟡 / 🔴 surfaces as below, and add a 🟡 offer to install `digest-composer`.

## 3d. Editor pass (protect the user's attention)

Before the final digest is assembled (after the members and the composer have produced candidate items), run the **digest editor** (`references/digest-editor.md`) over all candidate items from this run **plus yesterday's `config.carryover`**. It ranks by urgency×impact, caps each lens and the ✅ Handled section (top few; the rest carry over), collapses each issue to **one entry**, lists ✅ as **hyperlinked items with a brief "what changed" note** each, and enforces a hard cap of **2 to 3 proactive asks total** across gap checks, install offers (the bundled offer counts as one), and optional/adaptation nudges (respecting snooze). Items that don't make the cut are pushed to `config.carryover` for **tomorrow** (time-critical items are never deferred; they stay in-bucket or escalate). Every autonomous ✅ write gets a per-item audit note where the target supports comments (with previous value and revert hint) and **every** item (all buckets) is appended to the local **activity log**. This is what makes collapsing safe.

## 4. Compose the digest

One message to every enabled `digest.channels` entry (default: this chat), rendered per channel (`references/outputs.md`), in the user's `comms_profile` voice if set (**tone only**; never changes findings or actions). Title it with `config.digest.report_name` (default **Daily digest**) plus a one-line framing: *"Scanned your world since last run, setting aside what came straight to you."*

**Structure: the composer's categories.** The body is `digest-composer`'s composed output. Static categories, always shown: 💬 From the field & feedback → 📈 Adoption & usage → 🚢 Shipped & in-flight, with a **conditional 🧭 Competitive watch** below From the field & feedback and a **conditional ⚠️ Conflicts** above Shipped & in-flight, each only when it fires. Then the action surfaces. Readable full sentences; ≤3-minute ceiling; every actionable item deep-linked.

### 💬 From the field & feedback
New customer signal in the user's focus areas: customer calls, feedback/idea trends, support trends, sentiment. One issue = one entry, for example *"Three accounts asked in feedback this week to approve timesheets from their phones, and Initech raised the same ask on a call; it matches an existing idea."*

### 🧭 Competitive watch *(only when it fires)*
Relevant competitor releases in the user's areas, from `competitor-release-producer`. One issue = one entry, for example *"[A competitor added shared team calendars](url), which maps to a Calendar Sync feature you own."* Omitted entirely when nothing relevant is new.

### 📈 Adoption & usage
Recently-launched-feature adoption plus usage/analytics signal for the user's areas.

### ⚠️ Conflicts *(only when it fires)*
Collisions with others' work: the same problem or surface, or a crossed boundary (respect `config.radar.dismissed_conflicts`). Omitted entirely when there are none.

### ⚠️ Heads-up *(only when it fires)*
Something to watch that is not yet a collision or a blocker: engineering-progress `post_ship_regressions`, `cross_team_dependencies`, and customer-facing regressions to watch. Separate from Conflicts. Omitted when empty.

### 🚢 Shipped & in-flight
What shipped, moved, or is blocked (Jira), product-consequential decisions (engineering-progress `decisions`: a scope cut, a descoped epic, a bug or story closed Won't Do (or not planned, or canceled), or a committed item moved to a later release), plus feature maintenance. **Cross-reference** field & feedback where a fix addresses a pain, each pointing to the other.

### ✅ Handled for you
The autonomous work done this run (feature validation **Tier-A auto-validations**, publishes, and a `launch-watch` weekly re-pin, which is the dashboard link plus its one-line read), each a hyperlink plus a short "what changed" note. **Proof of work, not recap.** Omit if nothing was handled.

### 🟡 Needs your OK
Staged approvals that **aren't** discovery signal: chiefly feature validation's **Tier-B/C** features it believes are out of date (corrections drafted and staged, not written), plus install offers and gap checks. Each is a question with the specific action (*"Review 2 proposed edits"*), under the 2 to 3 proactive-ask cap; the rest carry over. Omit if empty.

### 🔴 Needs you to act
Only-you or blocking items no member can resolve: a connector re-auth, a missing required connector, a product-lead update discrepancy, engineering waiting on the user's answer (engineering-progress `needs_you`), a hard-blocked member. **Rare, but always shown when present; never folded away or capped out.** If one degraded today's run, also say so in the header (*"The support desk connection had expired, so there are no support trends today"*). Omit if empty.

### Set-aside tally
Close with one line: *"Set aside N already-seen items"* (and note if a surfaced trend was drawn from them). Offer *"show what I skipped"* on request.

**Rules:**
- **Discovery-signal actions live in their lens**, ordered last (never manufacture one). **Member approvals and only-you blockers are NOT discovery signal**; they go in the 🟡 / 🔴 surfaces, always shown when non-empty. Never drop a 🟡 or 🔴 to stay short: carry over 🟡 if over budget, but 🔴 is never dropped.
- **One issue = one entry**; **cross-link shared stories reciprocally** across lenses.
- **A clean lens is one reassuring line, never omitted.** ✅, 🟡, 🔴, and the tally omit only if truly empty.
- Gap-check, install, optional-skill, and adaptation asks attach to the **most relevant lens** when they relate to discovery signal, else to **🟡 Needs your OK**, still under the 2 to 3 proactive-ask cap.
- Keep it a **glance**; the editor pass enforces the caps.

### Quiet path
If nothing new was found, nothing was handled, **and** nothing needs the user: one line (*"All quiet: nothing new since your last digest."*) or, if `digest.quiet_days` is true, nothing. Never pad a lens to look busy.

## 4b. Adaptation pass (learning)

If `learning.enabled`, run the adaptation pass (`references/adaptation.md`) before persisting: append this run's behavioral observations; check whether any pattern is now stable (≥3× over ≥1 week); for each, add a 🟡 "I noticed … want me to …?" line (default), or, only if `learning.autopilot` is on and the change is low-risk and reversible, apply it and add a ✅ line saying how to undo it. Cap at 1 to 2 learning items per digest; respect `dismissed_patterns`. Record applied changes with provenance `set_by: learned`.

## 4c. Optional-skills nudge

Unless `config.optional_skills.stop_asking` is true, consider offering an unenabled optional member per `references/optional-skills.md`: when **a signal shows one would have helped** (for example a GA feature with no release note → `release-note-drafter`), **or**, when nothing signal-driven came up, as the **2-week backstop** once ≥14 days have passed since `optional_skills.last_offered`. Add it to 🟡, **cite the concrete activity since the last ask**, and always include the "…or tell me to stop offering these" opt-out.

## 4d. Resolve today's questions, then offer more (interactive)

When a human is present, follow `references/digest-editor.md` §5: present today's 🟡 items as questions (batched), **follow up** on any the buttons didn't resolve, and **only once every one of today's questions is resolved or explicitly deferred**, make the capacity offer: *"You're through today's questions; want to tackle {N} held-back items now, or leave them for tomorrow?"* On an unattended run, skip this; deliver the static digest and run the flow when the user next opens it.

## 5. Persist

Write `last_run` to `config.json`: ISO timestamp, `skills_ran`, counts per bucket, and any `blocked` members so the next run can re-surface unresolved 🔴 items until fixed. Also persist the updated `carryover` queue, any `snooze` changes, and confirm the run's items were appended to the activity log.

**If `config.json` can't be written** (no writable folder in this session): not a user-facing failure. Don't add a 🔴 item. If a human is present, offer to create a fresh config here (carrying over recoverable preferences) or show the updated config for them to save; if unattended, skip the write silently.

## Non-interactive rules (recap)
- No interactive questions on scheduled runs.
- Missing optional config → safe default + note. Missing required config for a member → skip that member + 🔴 line.
- Persist state so unresolved items carry forward rather than silently vanishing.
