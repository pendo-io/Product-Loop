---
name: daily-digest-conductor
description: 'Deprecated alias, kept for one release after this skill was renamed to daily-digest. It holds no run logic of its own. Load it only when an old scheduled task or a person names daily-digest-conductor explicitly; it then hands off to daily-digest, or stands down when the new daily task already exists so nobody gets two digests. Never choose it for a request that does not name it; use daily-digest.'
license: MIT. The repository's LICENSE has complete terms
compatibility: Needs the daily-digest skill from https://github.com/pendo-io/Product-Loop installed alongside it. Reading the scheduled-task list needs a client with scheduled tasks (for example Claude Cowork); without one it reads the config instead.
metadata:
  author: Pendo Product Operations
  version: "1.1.0"
---

# Deprecated: renamed to daily-digest

This skill was renamed to `daily-digest` in version 1.1.0. This copy exists only so an existing scheduled task, saved config, or habit that uses the old name keeps working for one release. It does no work of its own.

## What to do when invoked

1. **Stand down if the new task exists.** On a scheduled (unattended) run, check the scheduler's task list for `daily-digest-daily`, or read `<workspace>/.daily-digest/config.json` and check `schedule.task_id`. If either shows `daily-digest-daily`, exit without posting anything. The new task delivers today's digest; posting here would send a second one.
2. **Otherwise hand off.** If `daily-digest` is installed, follow it exactly as if it had been invoked with the same request and the same config. On an attended run, say once, first: *"daily-digest-conductor is now called daily-digest; running that."* Its first run migrates the saved config (with a backup) and offers to move the scheduled task to the new name.
3. **If `daily-digest` isn't installed**, run nothing. Attended: say the skill was renamed and offer to install `daily-digest` from https://github.com/pendo-io/Product-Loop. Unattended: post one line, *"Your daily digest skill was renamed to daily-digest and the new version isn't installed yet; install it to get your digest back."*

## Boundaries

- Never edit `config.json`, the activity log, or a scheduled task. `daily-digest` owns all three, including the rename migration.
- Never post a digest of its own.
- Remove this folder in the next release, once no saved config or task uses the old name.
