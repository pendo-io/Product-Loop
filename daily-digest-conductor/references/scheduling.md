# Scheduling (optional)

Load this only when registering or changing a schedule. Scheduling is optional: everything the conductor does also works on demand ("run my digest now"). When a scheduler is available, the conductor owns **one** recurring daily task and, optionally, companion tasks for members with a distinct native time.

## What you need

A client that can start a session on a schedule with a prompt, with the same connectors and the same workspace folder as your setup. Claude Cowork's scheduled tasks do this. Scheduled tasks in Cowork run on your machine, so **they fire only while the machine is awake and the app is open**; a missed run re-executes on next wake, and the daily run computes "due" from the real date so a late run is still correct. Any other scheduler works if it can run a Claude session with this skill, its connectors, and the workspace folder available.

## Registering the daily task

Prefer the programmatic path; fall back to guiding the user.

### Path A: programmatic (if the client exposes a create-scheduled-task tool)
Register:
- **task id:** `daily-digest-conductor-daily`
- **cadence:** daily at the confirmed local time (`schedule.time`, `schedule.timezone`).
- **prompt:** self-contained, for example:
  > Run the daily-digest-conductor daily run. Use the config at `{workspace}/.daily-digest/config.json`. Determine which enabled members are due today, run them non-interactively, and post the three-bucket digest (Done on your behalf / Needs your OK / Needs you to act) to this chat. Follow the skill's `references/daily-run.md`.
- **notify on completion:** on, so the digest surfaces.

### Path B: guided (no scheduling tool)
Walk the user through it; don't try to do it for them. In Cowork:
1. Open the project that has the workspace folder connected.
2. Type `/schedule` in the chat input.
3. Give it the task prompt above.
4. Pick **daily**, then the confirmed time and timezone.
5. Confirm. The task then appears in the scheduled-tasks list.

Surface these as a 5-step checklist in the setup wrap-up. For another scheduler, give the user the prompt and the time, and let them set it up.

### No scheduler
Say so plainly and move on: the user runs "run my digest now" each morning. The conductor still uses `last_run` for the "since last run" window, so a run after a gap covers the whole gap.

## Companion tasks (off-cadence members)

A member that needs to run at a time other than the morning digest (for example an afternoon change check) can get a companion task. None of the current members need one. `launch-watch`'s weekly refresh is not a companion task: it is a `weekly-<day>` cadence run by the daily task. Its dashboard's Generate button runs in the page and needs no task; re-pinning is a `launch-watch` run, so nothing for it is recorded in `schedule.companion_tasks`. If a future member's registry row declares a distinct native time, offer a second task `daily-digest-conductor-<member>` at that time, record it in `schedule.companion_tasks`, and register it only when a scheduler exists.

Keep companion tasks minimal. Weekly cadences are handled by the daily task's gating, not by separate tasks. Without a scheduler, the user can ask for that member by hand.

## Changing the schedule
1. Update `schedule.time` / `schedule.timezone` in `config.json`.
2. Re-register (Path A) or edit the task in the scheduler (Path B).
3. Keep config and the actual task in sync.

## Disabling (time off)
Toggle the task(s) off, or delete them, in the scheduler. The skill stays installed and invocable by name. Re-enable by toggling back on or re-registering.

## Testing without waiting
Say "run my digest now", or use the scheduler's "run now". Both execute the daily run immediately; a manual run may ask questions, unlike the scheduled one.
