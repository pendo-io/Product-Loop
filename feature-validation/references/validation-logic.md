# Validation logic

The decision procedure for turning "here's a due feature and here's what my sources say" into an action. Load this on every real run. This file is deliberately opinionated, the safety of unattended auto-validation depends on following it exactly.

## Scope preconditions (check before scoring anything)

Two kinds of feature are never candidates, regardless of tier or confidence:
- **Shipped features**: if the current phase is the shipped phase (`config.fields.shipped_phase`, default `GA`), the feature is shipped and out of validation scope. Drop it before research; never tier, stamp, or edit it.
- **Features the invoking user doesn't own**: only features where the invoking user is the listed owner (`config.fields.owner`) are eligible, and ownership is re-checked before any write. A non-owner never validates or edits a feature. (A pasted list counts as the PM's own.)

## The one rule everything else serves

**Absence of evidence is not confidence.** The agent may auto-validate a field only when it has found *positive, recent, authoritative* signal that the current recorded value is still true. "I searched and found nothing that contradicts it" is **not** corroboration, it is the *uncorroborated* state, and it caps the Feature at Tier B. Getting this wrong quietly poisons the feature records, which is the exact failure the whole system exists to prevent.

## Per-field classification

For each high-signal field (six by default, `config.high_signal_fields`), compare the current recorded value against the evidence gathered from that field's mapped sources (`config.field_source_map`). Classify into exactly one:

- **Corroborated**: a recent, authoritative source affirmatively states the same value (or a value that plainly means the same thing). Example: the record says `Regions available = Americas`; the launch brief updated this week lists Americas as the only region, and the release ticket names the same. Corroborated.
- **Contradicted**: a recent, authoritative source states a *different* value, or clearly implies the recorded value is wrong. Example: the record says `Next phase = Open beta`; the epic was re-planned yesterday to go straight to the shipped phase, and the roadmap doc now says the same. Contradicted.
- **Uncorroborated**: no source speaks to this field within a relevant recency window, or the only signals are stale/ambiguous/low-authority. This includes "I looked and found nothing." Uncorroborated is a *neutral* state, not a *positive* one.

### Recency weighting

- Prefer the **most recent** authoritative signal. A newer signal supersedes an older one.
- A signal older than the field's expected change frequency is **weak**: it corroborates weakly at best and cannot by itself carry a field to "corroborated" for Tier A purposes. (If the only thing confirming a target date is a planning doc nobody has touched for a quarter, treat the date as uncorroborated.)
- A **contradicting** recent signal always beats an older corroborating one → the field is contradicted.
- Authority order, high to low: system-of-record state (Jira status, Milestone target) > official announcement (launch post, GTM doc) > spec doc / conviction doc > informal mention (passing Slack comment). Don't let a low-authority mention outweigh a high-authority record.

## Feature tier (decision tree, in order)

Evaluate top-down; take the first branch that matches.

1. **Any field Contradicted, OR the agent believes any field should change → TIER C.**
   Percentage is irrelevant here. If research produced a specific belief that a value is wrong, even a strong hunch backed by evidence, it is Tier C, and the agent will propose the change. "Believes it's wrong" and "isn't sure it's right" are different states; only this branch proposes edits.

2. **No change believed necessary, AND all high-signal fields Corroborated, AND overall confidence ≥ 95% → TIER A.**
   Auto-validate. "All corroborated" is a hard prerequisite, one Uncorroborated field disqualifies Tier A even if everything else is rock-solid (**weakest-field gating**). The ≥95% is the residual-doubt bar on top of full corroboration: even with every field corroborated, if something about the corroboration is thin (single source, borderline recency, ambiguous phrasing) and overall confidence sits below 95%, drop to Tier B. The 95% is a judgment bar, not a computed score: there is no formula to run (see "Don't compute a fake numeric score" below).

3. **Otherwise → TIER B.**
   No believed change, but not fully/confidently corroborated. Present the record as-is and ask the PM to confirm. Propose nothing.

### Weakest-field gating, concretely

Feature confidence = the confidence of its *least*-corroborated high-signal field. Five fields fully confirmed and one field with no evidence = Tier B (or C if the agent thinks the date is actually wrong), never Tier A. This directly implements "index most on the high-signal fields": any one of them being shaky is enough to involve the human.

## What each tier does

| Tier | Tracker write during autonomous phase (tracker mode only) | Interactive step | `Validated by` | Proposes edits? |
| --- | --- | --- | --- | --- |
| A | `Last Validated` = today + `Validated by` = Agent + audit comment | none | Agent | no |
| B | none (staged) | PM confirms as-is | User | **no** |
| C | none (staged) | PM + agent revise, then validate | User | yes |

- **Tier A** is the only branch that completes without a human, and only after the one-time calibration run in tracker mode (see `SKILL.md` step 7). In markdown mode Tier A is reported as validated by Agent with its audit line, and nothing is written.
- **Tier B** shows current values + names uncorroborated fields, but suggests no changes. A "no" from the PM converts it to Tier C in place.
- **Tier C** never validates until agent *and* PM agree the record is accurate.

## Audit comment format

Every validation (Tier A auto, or Tier B/C after the PM) gets an audit line. In **tracker mode**, add it as a comment on the feature record (or, if the tracker has no comments, in a notes field the PM named at setup; never create one). In **markdown mode**, print it under the feature in the report so the PM can paste it. This is what makes autonomous validation inspectable after the fact.

Some trackers render block-level markdown in comments (code fences, bullets, headings) as plain text; Notion does. So write the audit as short plain lines separated by newlines, using inline bold for field names, and don't rely on bullets or fences. Cite sources by name and link; put ticket numbers only inside URLs. The examples below show the intended text (the surrounding ``` is just for this doc).

**Tier A (auto):**
```
✅ Auto-validated <today> by the Feature Validation agent.
Corroborated. Current phase (Closed beta): epic status (<ticket URL>). Next phase (Open beta): Quarterly plan. Next phase target (<date>): milestone "<milestone name>". Uses AI? (No): spec doc scope section. Regions available (Americas): Release checklist. Early access details: Beta program FAQ.
No contradicting signals found.
```

**Tier B (user-confirmed):**
```
✅ Validated <today> by <PM name> (confirmed as-is).
Agent could not independently corroborate: Uses AI?. PM confirmed the record is accurate.
```

**Tier C (user-revised):**
```
✅ Validated <today> by <PM name> (revised then confirmed).
Changed. Uses AI?: No → Yes (spec doc approach section, <spec doc URL>). Other fields confirmed accurate.
```

If the tracker supports date mentions in comments (Notion's `<mention-date start="YYYY-MM-DD"/>`, for example), use one for the date.

Keep it factual. The audit line is an audit trail, not prose.

## Worked examples

- **Every field is backed by fresh, authoritative sources, nothing disagrees, and you would sign off without review (clearly past the 95% bar) → Tier A.** Validate, add the audit line, move on. The PM doesn't need to see it.
- **Every field confirmed except `Early access details`, which no recent source mentions → Tier B.** Show the record unchanged, name the field you couldn't confirm, and ask "still accurate?" Propose nothing: not being able to confirm a value gives no basis for a new one.
- **The roadmap doc and the epic both moved the target date out by a month, but the record still has the old date → Tier C.** Propose the new date with both sources, agree it with the PM, then validate as User.
- **The record says `Regions available = Americas`, but the launch brief now also lists APAC → Tier C**, however solid the other fields are. One field you believe is wrong is enough.
- **The feature is due and none of the PM's sources mention it → Tier B.** Not Tier A, since nothing corroborates it, and not Tier C, since there's no specific change to propose. Show it and ask. If the same feature keeps landing here, it becomes a coverage-insight candidate (`references/sources.md`).

## Confidence is a judgment, not a formula

Don't compute a fake numeric score. The ≥95% bar is a *disposition*: "I would be comfortable telling the PM I confirmed this without their review, and standing behind it." If you'd hedge, it's not ≥95%. When genuinely torn between Tier A and Tier B, choose Tier B, a needless confirmation costs the PM ten seconds; a wrong auto-validation costs the feature records their trustworthiness.
