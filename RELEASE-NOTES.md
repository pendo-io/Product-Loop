# Release notes

## 0.2.0

### Two daily-digest skills have new names

The old names made it hard to tell which skill did what: the conductor that runs your daily digest was called `daily-digest-conductor`, and the skill that writes it was called `attention-radar`. They now match their jobs.

| Old name | New name | What it does |
|---|---|---|
| `daily-digest-conductor` | `daily-digest` | Runs your daily skills and posts the digest |
| `attention-radar` | `digest-composer` | Writes the digest from the producers' output |

**What happens when you update**

- **Your settings move over on their own.** The first `daily-digest` run renames the skills in your saved config. It saves a backup first, `config.pre-rename.bak.json` (and `activity-log.pre-rename.bak.jsonl`), in the same `.daily-digest/` folder. Later updates never overwrite these two files. The folder, your report name and your digest settings stay as they are.
- **The old names still work for this release.** `daily-digest-conductor` and `attention-radar` are now short stubs that hand off to the new skills. They will be removed in the next release.
- **You still get one digest a day.** Your existing scheduled task keeps working through the stub until you move it. Once a `daily-digest-daily` task exists, the old task stays silent.

**What to do**

1. Update the plugin.
2. Move your scheduled task. On your next run with you present, `daily-digest` offers to do this for you: it registers `daily-digest-daily` at your usual time and then removes `daily-digest-conductor-daily`. If you'd rather do it by hand, create a new daily task with the prompt in `daily-digest/references/scheduling.md`, then delete the old `daily-digest-conductor-daily` task.
3. If you call either skill by name, use the new name.

If anything looks wrong after the update, restore `config.pre-rename.bak.json` over `config.json` in your `.daily-digest/` folder.

The `competitor-release-producer` and `support-trends-producer` install checks accept the composer under either name, so a setup that hasn't updated every skill yet keeps working.
