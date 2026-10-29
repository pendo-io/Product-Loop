# Scheduling (optional)

Load this only when registering or changing a schedule. Scheduling is optional: everything the conductor does also works on demand ("run my digest now"). When a scheduler is available, the conductor owns **one** recurring daily task and, optionally, companion tasks for members with a distinct native time.

## What you need

A client that can start a session on a schedule with a prompt, with the same connectors and the same workspace folder as your setup. Claude Cowork's scheduled tasks do this. Scheduled tasks in Cowork run on your machine, so **they fire only while the machine is awake and the app is open**; a missed run re-executes on next wake, and the daily run computes "due" from the real date so a late run is still correct. Any other scheduler works if it can run a Claude session with this skill, its connectors, and the workspace folder available.

## Registering the daily task

Prefer the programmatic path; fall back to guiding the user.

### Path A: programmatic (if the client exposes a create-scheduled-task tool)
Register:
- **task id:** `daily-digest-daily`. Record it in `schedule.task_id`.
- **cadence:** daily at the confirmed local time (`schedule.time`, `schedule.timezone`).
- **prompt:** self-contained, for example:
  > Run the daily-digest daily run. Use the config at `{workspace}/.daily-digest/config.json`. Determine which enabled members are due today, run them non-interactively, and post the three-bucket digest (Done on your behalf / Needs your OK / Needs you to act) to this chat. Follow the skill's `references/daily-run.md`.
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

A member that needs to run at a time other than the morning digest (for example an afternoon change check) can get a companion task. None of the current members need one. `launch-watch`'s weekly refresh is not a companion task: it is a `weekly-<day>` cadence run by the daily task. Its dashboard's Generate button runs in the page and needs no task; re-pinning is a `launch-watch` run, so nothing for it is recorded in `schedule.companion_tasks`. If a future member's registry row declares a distinct native time, offer a second task `daily-digest-<member>` at that time, record it in `schedule.companion_tasks`, and register it only when a scheduler exists.

Keep companion tasks minimal. Weekly cadences are handled by the daily task's gating, not by separate tasks. Without a scheduler, the user can ask for that member by hand.

## Handing over from the old task id

Before 1.1.0 the daily task was registered under the conductor's old name (see the map in `references/updates.md` → "Renamed skills"). Its prompt invokes the old name, which is now a deprecated stub that forwards to this skill, so an old task keeps producing the digest until it is handed over. Two rules keep it to one digest a day:

- **The stub stands down.** When the stub runs unattended and finds `daily-digest-daily` in the scheduler's task list, or `schedule.task_id` set to `daily-digest-daily`, it exits without a digest.
- **The handover happens on an attended run**, after the config migration and with the user's OK, because it changes a scheduled task:
  1. List scheduled tasks. If an old-id task exists, offer one 🟡: *"Your daily task still uses the old name. Want me to move it to `daily-digest-daily` at the same time and remove the old one?"*
  2. On a yes, register `daily-digest-daily` (Path A) with the old task's time and timezone and the prompt above, set `schedule.task_id`, and list tasks again to confirm the new one exists.
  3. Only then delete or disable the old task, and list tasks once more to confirm exactly one daily digest task remains. If the old task can't be removed, say so and leave the stub's stand-down rule to keep it silent.
  4. Path B (no scheduling tool): give the user the new prompt and the two steps (create the new task, delete the old one), and set `schedule.task_id` once they confirm.
- **On a no**, keep the old task and record `schedule.task_handover_declined: true` so the offer doesn't repeat daily. The old task keeps working through the stub until the stub is removed in a later release; ask again then.

## Changing the schedule
1. Update `schedule.time` / `schedule.timezone` in `config.json`.
2. Re-register (Path A) or edit the task in the scheduler (Path B).
3. Keep config and the actual task in sync.

## Disabling (time off)
Toggle the task(s) off, or delete them, in the scheduler. The skill stays installed and invocable by name. Re-enable by toggling back on or re-registering.

## Testing without waiting
Say "run my digest now", or use the scheduler's "run now". Both execute the daily run immediately; a manual run may ask questions, unlike the scheduled one.
