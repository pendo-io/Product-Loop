# Sources, coverage logging, notifications, and insights

Load when reading a source, logging coverage, sending the run notice, or surfacing a coverage insight. Every source here is optional. Tool names vary by connector; check the connected tool's schema if a call misbehaves.

## Reading each source (for corroborating the high-signal fields)

Read only what `field_source_map` points to, for the features actually due this run; never scan the firehose. For each due feature, gather evidence targeted at its high-signal fields.

### No connectors

If no source connectors are available, use what the PM pasted: links, excerpts, ticket statuses, launch posts. A field with no pasted or fetched evidence is **uncorroborated** (Tier B at best). At the end of the run, say which sources would let more features reach Tier A.

### Jira or another issue tracker (Current phase, Next phase, target date)

Phase and status usually live in the linked epic or issue. If the feature record stores a ticket key or URL, fetch that directly. Otherwise prefer a targeted search over enumerating a project, for example JQL:

```
text ~ "<feature name>" ORDER BY updated DESC
```

narrowed by `config.sources.jira.project_keys` if set, max 10 results. Then read status, fix version, due date, and the recent changelog. If a call fails, try the connector's generic search or fetch tool, then skip and mark those fields uncorroborated for this run.

### The feature's spec doc (Uses AI?, Regions, EA details)

The spec link on the feature record (`config.fields.spec_link`) is the richest source for the non-phase fields; a slide or secondary link field is a backup. The feature's **conviction doc**, if linked, is an additional external source for the same fields; read it the same way. Also read the adjacent notes fields (AI, region notes) for context; as same-record fields they inform understanding but don't count as *external* corroboration. Resolve what the link points to:
- **Google Drive** → read the file content; check its modified time first to weight recency.
- **Confluence** → read the page as markdown.
- **Notion** → fetch the page.
- **Anything else** → fetch if a tool can; otherwise ask the PM to paste the relevant section.

### Watched docs (roadmap, GTM briefs, EA docs)

From `config.sources.doc_sources`, using the same fetchers by `type`. For a Drive folder, search for files modified since the last run. For a Confluence space, search the space for the feature name, for example CQL:

```
space = '<spaceKey>' and text ~ '<feature name>' and lastmodified > '<window_start YYYY-MM-DD>'
```

### Slack (launch announcements, EA chatter)

From `config.sources.slack_sources`. Prefer targeted search over reading a whole channel: search for `"<feature name>"` since the window start, and narrow with `in:#channel`, `from:@user`, `has:link`. Weight an official launch post far above a passing comment (authority order in `validation-logic.md`).

### Recency window

Validation cares about the *latest* state, not a fixed slice. Pull the most recent authoritative signal per field regardless of age, but apply recency weighting: a signal older than the field's expected change frequency is weak. For phase and target, "recent" means roughly within the current or last cadence period; for AI (rarely changes), an older spec doc still corroborates if nothing newer contradicts.

## Coverage logging

Skip this unless a state file is kept (saved config). Each time a field on a feature is classified, add one entry to `state.json.coverage_log`; the entries are the audit trail and the input to coverage insights.

Entry shape:
```json
{
  "run": "<ISO 8601 timestamp>",
  "feature_url": "<feature record URL>",
  "feature_name": "Plan & Track",
  "field": "Early access details",
  "outcome": "uncorroborated",
  "source": null,
  "tier": "B"
}
```

`outcome` is `corroborated`, `uncorroborated`, or `contradicted`. `source` names the source when corroborated (for example `jira:<ticket URL>`), `null` when none.

**Rotation (keep the log bounded):** keep a running total per field in `state.json.coverage_counters`, shaped like `{ "<field>": { "corroborated": <n>, "uncorroborated": <n>, "contradicted": <n>, "features_seen": <n> } }`. These totals are never thrown away. The raw `coverage_log` keeps only its newest ~200 entries. Add to the totals with every new entry, and trim the raw log once the run ends.

## Notify (the run notice)

Optional; it needs Slack or another messaging connector. `config.notify` controls it with three keys: `slack` (true or false), `mode` (`always`, `only_if_tier_c` or `never`) and `slack_destination_user_id`. When Phase 1 ends and a notice is due, send the PM a DM:

```
Feature validation finished: <N> validated, <M> to confirm, <K> to revise. Pre-change items are at the top. Open the run's chat and reply 'go' when you're ready.
```

Send only when there's something for the PM (`M + K > 0`) unless `mode == always`. If `mode == only_if_tier_c`, send only when `K > 0`. Never send on a nothing-due run. If a run failed on auth, send the failure instead (see `SKILL.md` gotchas).

## Coverage insights (Tier B → Tier A)

Every Tier B is residual manual work. Where Tier B clusters on a *field*, there's usually a fixable coverage gap. Surface it sparingly and specifically.

**When to surface:** at the end of an interactive session (never during the autonomous phase, never on a nothing-due run), check `state.json.coverage_counters`. A field is a candidate when, over the tracked window, its `uncorroborated / features_seen` ratio exceeds `config.coverage.insight_threshold` (default `0.5`) **and** `features_seen >= config.coverage.min_features` (default `4`). Surface **at most one** insight per session, the most acute candidate. Without a state file, a single run can still note one pattern it saw in that run, framed as an observation.

**Distinguish the two kinds of gap:**
- **Source exists, agent isn't reading it.** If, on the times the field *did* corroborate, the corroboration came from a source not in `config.sources` (visible in `coverage_log.source`), recommend adding it: *"I couldn't confirm `Early access details` on most features this month. The few times I could, the answer was on your beta signup page, which isn't in my sources. Add that page and these should start clearing on their own."*
- **No artifact exists anywhere.** If the field essentially never corroborates from any source, it's a process gap: *"None of your sources say whether a feature uses AI. A one-line note in each spec doc would let me confirm `Uses AI?` without asking you."*

**Never nag:** record declined recommendations in `state.json.declined_insights` (keyed by field + kind). Don't re-raise a declined one unless the PM asks to revisit coverage.

**Acceptance:** if the PM accepts a "connect a source" recommendation, run the source step in `references/setup.md` to add it, then confirm it takes effect next run.

This is per-PM. The counters are structured so a later cross-PM rollup is a straightforward aggregation.
