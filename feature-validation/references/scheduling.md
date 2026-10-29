# Scheduling (optional)

Load only when registering or changing a schedule. The skill works fully on demand; a schedule only saves the PM from asking.

## Where scheduled runs work

Any host with scheduled tasks can run this skill. Claude Cowork (the desktop app) is one: a scheduled task runs as its own session, and **that session persists**, so the PM opens it later and continues the conversation. That's how the interactive phase (resolving Tier B/C) works. Other hosts with scheduled or cron-style agent runs work the same way if the run's chat, or `state.json`, can be picked up later.

Constraint (Cowork): scheduled tasks run only while the computer is awake and the app is open. Missed runs re-fire when the machine wakes, so the run must read **today's actual date** rather than assuming when it fired.

## Setup steps (Cowork example), surface as a checklist

The skill can't register the task itself; the host requires the user's explicit consent to create a recurring task. Once the config is saved:

1. Open the Cowork project that points at the workspace folder holding the config (or create one).
2. In the chat input, type `/schedule`.
3. When asked for the task instructions, paste:

   ```
   Run the feature-validation skill for me. It reads its config from the
   feature_validation section of config.json in this workspace.
   ```

   If you use the `daily-digest` conductor, write `.daily-digest/config.json` in place of `config.json`, since that's where this skill's section lives.

4. Pick **daily** (recommended) and a time the machine is usually awake. Mid-morning is a good default so the PM can resolve staged items during the workday.
5. Confirm. The task appears under scheduled tasks.

## Testing without waiting

Say "run feature validation now" in the project chat for a one-off run, or use the scheduled task's run-now option. Use this for the calibration run.

## Changing or pausing

Edit or delete the task in the host's scheduled-task list, then update `config.schedule` to match. Pausing the task leaves the skill installed and runnable by name.

## Non-interactive behavior on scheduled runs

- **Autonomous phase:** don't ask questions; nobody may be watching. Do the Tier A actions (post-calibration in tracker mode), stage Tier B/C to `state.json`, end with a resumable prose summary, and send the notice per `config.notify`.
- **Interactive phase:** later, in the same chat, when the PM responds. Plain conversational confirmation for the per-feature walk-through.
- Always write `state.json` and `config.last_run` before ending the autonomous phase, so a resume (even days later, or after a context reset) can reload the pending queue.

## Auth caveat

Scheduled runs can't open browser re-auth. If a connector throws an auth error, catch it, notify the PM ("Feature validation couldn't run, reconnect <tool>"), write `last_run.status = "failed_auth"`, and exit. Don't silently retry.
