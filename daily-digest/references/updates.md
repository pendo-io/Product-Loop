# Updates: absorbing a newer conductor or member without clobbering the user

Load this when a newer version of the conductor or of a member is installed, or when the user says "update my digest setup", "here's a new version of {skill}", or "add this new skill". The goal: **incorporate what the new version requires, adopt safe default changes the user never touched, and preserve every customization**, asking only where there's real ambiguity.

## Why provenance matters

Every config write records provenance in `_meta.provenance.<path>`: `set_by` (`spec_default` | `user` | `inferred` | `learned`), `base_value` (the default at the time), and `spec_version`. That gives three versions of each setting:

- **base**: the default when the user last synced (`base_value`);
- **current**: the value in the live `config.json`;
- **incoming**: the default or requirement in the new version.

Without a base you can't tell a user edit from a changed default, and you'd either clobber the user or ignore the update. Always record provenance.

## What the new version requires

Read, in order: a `## Requirements` section in the incoming skill (connectors, config keys, source types); else infer from its setup section. Anything inferred as required that would change user data is treated as ambiguous and asked, not auto-written.

## Classify each setting

| Case | Condition | Action |
|---|---|---|
| **A. Newly required** | The new version needs a key, connector, or source type the current config lacks | **Incorporate.** Default or infer it if possible; otherwise **ask**. A newly required connector becomes a 🔴 "connect X" line, not a block. |
| **B. Safe refinement** | The default changed AND current == base (the user never touched it) | **Adopt** the new default silently. |
| **C. Protected customization** | both current and incoming differ from base | **Keep the user's value.** Bump `base_value` to incoming so it isn't flagged again. For a *preference* (run time, channels, review step, quiet days, cadence, tone), optionally add one soft "you might want to try…" note, never a change. |
| **D. User customization, default unchanged** | current != base AND incoming == base | **Keep the user's value.** |
| **E. Removed or renamed field** | The new version drops or renames a field the user set | Migrate a clean rename and note it. Otherwise **ask** whether to drop or keep it. Never silently delete user data. |
| **F. User-added** | Extra sources, channels, or members the user added | **Always preserve.** The spec is a floor, not a ceiling. |
| **G. Shape change** | A field's type changed and the user's value no longer fits | Migrate if unambiguous; otherwise **ask**. |

**New source types**: prompt, never auto-enable and never ignore (see `references/sources.md`).

## Removals and core changes are never silent

- **A member the user enabled is gone or renamed** in the catalog: tell the user what changed and why (from the release notes, if any), keep their config, and offer: keep using their installed copy, switch to the replacement if one is named, or retire it (config kept in a restorable backup). Default: keep using it.
- **A capability the user relies on was cut**: flag it with the reason, keep any config that fed it, and offer a migration if the capability moved elsewhere.
- **A core change to how the conductor behaves** (the digest model, the run flow): lead the report with a short before/after in plain English and ask the user to confirm adoption; offer rollback (`config.bak.json`).

On an **unattended** run, apply only non-destructive changes and defer every removal and core-change confirm to the next attended run.

## Renamed skills (1.1.0)

Version 1.1.0 renamed two skills. This is the migration map; it is the only place the conductor matches the old names.

| Old name | New name | Old copy after the update |
|---|---|---|
| `daily-digest-conductor` | `daily-digest` | Deprecated stub that forwards here; removed in a later release |
| `attention-radar` | `digest-composer` | Deprecated stub that points here; removed in a later release |
| task `daily-digest-conductor-daily` | task `daily-digest-daily` | Retired at the next attended run (below) |
| task `daily-digest-conductor-<member>` | task `daily-digest-<member>` | Same as the daily task |

Run it on the first run that finds an old name in `enabled_skills[].name`, an `installed_skills` key, a `members` value, `last_run.skills_ran`, or `schedule.companion_tasks`, or finds `_meta.conductor_version` below 1.1.0. It is a clean rename (case E), so it runs on an unattended run too.

1. **Back up first, to the rename's own files.** Copy `config.json` to `config.pre-rename.bak.json` and `activity-log.jsonl` to `activity-log.pre-rename.bak.jsonl`, then re-read both copies. If a file with that name already exists, keep it and don't overwrite it: it holds the config from before the first migration attempt. These names are separate from `config.bak.json`, which every later update overwrites. If either backup can't be written, stop: change nothing and raise one 🔴 line saying the update is waiting for a writable config home.
2. **Rename in `config.json`**, using the map: `enabled_skills[].name`, the `installed_skills` keys, `members` values, `last_run.skills_ran`, `carryover.items[].skill`, `schedule.companion_tasks`, and `_meta.provenance` keys that contain a skill name. If both the old and the new key exist in `installed_skills`, keep the new entry and carry over `mute_reminder: true` from the old one. Leave cadence, status and every other value as they are.
3. **Set `members.composer` to `digest-composer`**, unless the user pointed it at some other skill (case F: keep theirs).
4. **Rename the `skill` field in past activity-log rows** so usage history and the adaptation module see one name per skill, then append one row recording the migration (`bucket: "done"`, `skill: "daily-digest"`, summary "renamed skills after the 1.1.0 update", `revert_hint: "restore config.pre-rename.bak.json and activity-log.pre-rename.bak.jsonl"`).
5. **Don't touch** the config home `.daily-digest/` (every user's config path depends on it), the `radar` block (it isn't keyed by skill name), `competitor_radar`, or `digest.report_name`, even when it reads like an old skill name: the user chose it.
6. **Bump `_meta.conductor_version` to `1.1.0`**, record provenance, and re-read `config.json`.
7. **Hand over the scheduled task** (`references/scheduling.md` → "Handing over from the old task id"). On an unattended run, skip this step: the old task keeps working through the stub, so the user still gets exactly one digest. Every later attended run offers the handover until it's done or declined (`references/daily-run.md` step 1), so it doesn't depend on this migration running again.
8. **Report it once** in the next digest, under ✅: the two new names, the backup files, and, if the task isn't handed over yet, a 🟡 asking to do it.

Running it twice is harmless: the second run finds no old names and stops after step 1's check.

## Apply, ask, report

1. **Back up** `config.json` to `config.bak.json` first.
2. **Apply** cases A (auto), B, C (base bump only), D, F, and unambiguous E/G migrations. Update provenance and `_meta.conductor_version`.
3. **Ask** every remaining decision in **one** batch, recommending the safer option first.
4. **Report** in groups, omitting empties: *Incorporated (required by the update)* · *Adopted (defaults you hadn't changed)* · *Kept your customizations* · *You might want to try* · *Needs your call*. Mention the backup.
5. **Offer sources**: ask if there's anything new to watch, and recommend the few highest-value missing sources the updated members lean on (opt-in; record declines).

## Worked example

A newer conductor raises the default `digest_budget.show_per_bucket` from 3 to 4 and lowers `digest_budget.max_proactive_asks` from 3 to 2, and a newly installed `support-trends-producer` needs `org_constants.support_desk_domain`.

- `show_per_bucket` is still 3 and was never edited (current == base) → **Case B**: adopt 4, report it under "Adopted."
- The user had set `max_proactive_asks` to 1 (current ≠ base 3, incoming 2 ≠ base) → **Case C: keep 1**, bump base to 2, report "kept your limit of one proactive ask."
- The support desk domain can be read from a support source already in `config.sources` → **Case A auto**: write it, report it.
- If no source gives it → **Case A ask**: one question, "support-trends-producer needs your support desk domain to build ticket links; what is it?"
