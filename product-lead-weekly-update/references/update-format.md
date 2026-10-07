# Update format

Load this when drafting the rollup. Don't load it during setup.

## The default template

Five conceptual prompts:

- **Headline**: a 1 to 2 sentence summary of the week's overall state
- **Feature / initiative updates**: brief status per active workstream (what changed, key learnings, impact on metrics, what's next)
- **Customer commitments**: named customer commitments and their current status. Only if the team tracks them (`config.template.tracks_customer_commitments` is true); otherwise skip this prompt
- **Risks & blockers**: anything threatening timelines or needing escalation
- **Coming up**: key milestones or deadlines in the next 1 to 2 weeks

In practice, leads **don't** ship rollups with those five headers
literally. They cover the dimensions across the body in their own
structure. Do the same. If `config.template.source` is `match_existing`,
mirror the reference rollup; if `custom`, follow
`config.template.custom_notes`.

## Title format

**Fixed. Always use this, regardless of the lead's prior titles:**

```
<Lead name> Update - <YYYY-MM-DD>
```

The date is the date of the rollup (the run day on a schedule). In
markdown mode it's the `#` heading. In tracker mode it's the record title,
with inline person and date mentions if the tracker supports them (see
`tracker-mode.md`). Do **not** mirror historical titles: they tend to be
inconsistent plain text ("<Name> Update - <month>/<day>", "... Rollup"), and
spelled-out dates aren't sortable. Mirror the lead's prior **body**
structure only.

## Summary

A separate at-a-glance summary of the rollup:

- 3 to 6 short bullet sentences, each starting with `- `
- Each bullet is a self-contained metric, milestone, or decision
- No formatting or links inside the bullets; those go in the body

In markdown mode, put it under a `**Summary**` heading right after the
title. In tracker mode, write it to the tracker's plain-text summary
property if one exists (see `tracker-mode.md` for line breaks); otherwise
put it at the top of the body.

Example:

```
- Workload View forecasting is open to every beta account, and weekly forecast views doubled once the Monday email linked to it.
- Timesheets approval reminders shipped, and overdue approvals dropped in the first week.
- The team dropped custom intervals from the first Recurring tasks release, which brings its beta forward one sprint.
- The Calendar Sync Outlook beta is waiting on a legal review of the consent screen, with no date yet.
- Leadership input needed on whether Plan & Track text alerts launch on the top plan first or wait to launch on every plan.
```

Draft the body first, then write the Summary as a compression of it.

## Structural patterns for the body

After reading the lead's last 2 or 3 rollups, classify which pattern they
use and mirror it.

### Pattern A: structured

Bolded initiative header per initiative. Sub-bullets use a fixed
layout (progress towards key metrics, learnings, milestone progress,
blockers). That layout is how a structured lead lays out each initiative;
it is not the five template prompts, which are coverage across the whole
rollup. Links inline as markdown. Headers are plain unless the lead's
prior rollups put emoji in them; then keep the emoji (see Falling back).

```
**Workload View forecasting**
- **Progress towards key metrics:** Weekly forecast views doubled after the Monday email started linking to the Forecast tab. The goal is a third of active teams opening it each week.
- **Learnings:** Agency teams say four weeks is too short to plan with, so the team is trying an eight-week view with a few beta accounts.
- **Milestone progress:** Open to every beta account; planning for the general release starts next sprint. [Planning doc](<url>).
- **Blockers:** None.

**Calendar Sync for Outlook**
- **Progress towards key metrics:** None yet, since the beta can't open until the consent screen is approved.
- ...
```

Use this pattern for leads who already do this. Never invent it for a lead
whose prior format is prose.

### Pattern B: prose

Bolded initiative name as an inline lead-in, then a free paragraph.
Initiatives stack vertically. No formal sub-bullets. Emoji only where the
lead's prior rollups use them.

```
**Recurring tasks**: We cut custom intervals from the first release after usability sessions showed almost everyone repeats work weekly or monthly. That pulls the beta forward a sprint. Jordan is writing the in-app note that tells people custom intervals are still coming.

**Timesheets**: Approval reminders went out to everyone this week. Early signs are good: fewer approvals are sitting overdue, and the only support questions so far are about changing the reminder day.
```

### Falling back

If the lead has fewer than 2 prior rollups, use
`config.style.fallback_style` (default: structured, Pattern A, which is
more forgiving for a first draft).

Headers on a first rollup, with no prior rollup to mirror, are plain
bolded section headers with no emoji. Once prior rollups exist (even
one), mirror their header style: emoji if they use emoji, plain if they
don't. Never add emoji the lead doesn't use.

## Content coverage

Across the body, each initiative should touch on:

- **Theme-level progress**: what happened across the team, not what each person did line by line. Readers can open the PM updates for detail.
- **Cross-cutting risks**: things spanning several PMs, features, or teams.
- **Decisions made and needed**: what got decided this week, and what needs leadership input.
- **Customer signals**: named customers, escalations, and commitments (commitments only if the team tracks them).
- **What's next**: concrete milestones and dates in the next 1 to 2 weeks.

## Absent-PM handling

Per `config.style.absent_pm_handling`:

- **`mention`**: a one-line note inline near the relevant initiative: "No update from <PM> this week."
- **`omit`**: leave it out. The initiative still appears if other context covered it.
- **`footer`**: one line at the end of the body: `*Awaiting updates from: <PM A>, <PM B>.*`

Default to `footer`; it's the lowest-friction option for the lead and the
reader.

Summary: an absent PM goes in the Summary only when the absence hides a
known risk (for example, the PM's feature has an open escalation or a
date at risk, and without their update the rollup can't say where it
stands). Otherwise keep the Summary about the work and handle absence in
the body only.

## Discrepancy callouts

When the synthesis finds a real disagreement between PM updates and
supplementary context, add a callout near the relevant initiative:

```
⚠️ Possible discrepancy: Jordan reports the Outlook beta opening next week, but Thursday's legal sync note says the consent screen review hasn't started. Worth checking.
```

Rules:

- Don't try to resolve it. The lead decides.
- At most **2 to 3 callouts** per rollup. If you're finding more,
  something is wrong with the data or the comparison; say so in the review
  note instead of cluttering the rollup.
- Don't flag trivial mismatches (different phrasing of the same status,
  small date differences within the same week).
- Record them in `last_run.flagged_discrepancies`.

## Length

Body: 200 to 600 words. Summary: 3 to 6 bullets. A rollup is
summarization, not new content; past 700 words you're probably
re-narrating the PM updates instead of synthesizing.

## Drafting checklist before delivering

- [ ] Title is exactly `<Lead name> Update - <YYYY-MM-DD>` (not mirrored from prior titles)
- [ ] Summary is 3 to 6 plain-text bullets, no formatting or links
- [ ] Tracker mode: product-area relation set to every area in `config.team.product_areas`; computed fields not in the payload
- [ ] Headers mirror the lead's prior rollups, emoji or not; plain headers with no emoji if there are none
- [ ] Body covers the 5 template prompts conceptually (4 if the team doesn't track customer commitments), in this lead's prior format
- [ ] Absent PMs handled per config; in the Summary only if the absence hides a known risk
- [ ] Discrepancy callouts (3 at most) inserted where applicable
- [ ] Body is 200 to 600 words
- [ ] Nothing invented: every claim traces to a PM update or a source read this run

If an item fails, fix the draft and run the checklist once more. If an item still fails after that second pass, deliver the draft and name the failing item in chat; don't loop.
