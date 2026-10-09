# Update format

Load this when drafting the body of the weekly update. Don't load it during
setup.

## The default template

The default template is four prompts:

- Commentary on milestone progress
- Update on progress toward key metrics and any learnings / adjustments to selected metrics
- What you're hearing qualitatively
- Blockers and progress toward next delivery

In practice, PMs do **not** ship updates with those four headers. They
treat them as conceptual coverage and write in their own structure. Do the
same: cover those four dimensions across the body, but format per the PM's
actual prior style.

If the feature record shows its milestone or metrics fields and they're
empty (or the user says the feature has none), don't invent commentary
for those prompts: write "no metrics set" in that feature's bullet
instead. When that isn't known, as in a paste-only run with no feature
record, skip the prompt silently. A feature whose only activity is customer feedback keeps
its single feedback line (see Content coverage) and needs no extra note.

If `config.template.source` is `match_existing`, mirror
the linked or pasted reference update; if `custom`, follow
`config.template.custom_notes`.

## Title format

Match this exactly:

```
<PM name> - PM update - <YYYY-MM-DD>
```

The date is the date of the update (the run day). No leading dash, no other
text. In markdown mode it is the document's `#` heading. In tracker mode it
is the record title; if the tracker supports inline person and date
mentions (Notion does), use them for the name and date, because they make
the records sortable and groupable. See `references/tracker-mode.md`.

**This title format is fixed.** It is *not* subject to the "mirror the PM's
prior style" rule below; that rule governs **body structure only**.
Historical titles vary (some plain text with spelled-out dates, some
without "PM", some with a second date); do **not** copy those.

The product-lead-weekly-update skill finds PM updates by this title, so
keep it exact.

## Features covered line (markdown mode)

End the markdown update with one line listing every feature the body
covers, in the order they appear:

```
**Features covered:** <Feature A>, <Feature B>, <Feature C>
```

This replaces the tracker's feature relation in markdown mode, and the
product-lead skill reads it. In tracker mode, set the tracker's feature
relation instead (same list), and omit the line.

## Structural patterns to mirror

After fetching the PM's last 2 or 3 updates, classify which of these
patterns they use. Mirror it.

### Pattern A: grouped by product area

Top-level bullet per product area. Sub-bullets per feature, each starting
with the feature name and a colon, followed by 1 to 3 narrative sentences.

```
- Workload View
    - Forecast tab: Beta feedback says four weeks is too short for agency teams, so we're trying an eight-week view with a few beta accounts.
    - Team grouping: Design approved on Tuesday. The engineering estimate comes back Wednesday.
    - Drag to reassign: The undo bug QA found is fixed and goes out in the next release.
- Calendar Sync
    - Outlook two-way sync: Waiting on legal to approve the consent screen copy before the beta can open.
```

Use this when the PM owns features that cluster cleanly into 2 to 4
product areas.

### Pattern B: flat per-feature

Top-level bullet per feature, each starting with the feature name (linked
to its record if one exists), then 1 to 3 narrative sentences covering
progress, blockers, decisions, and what's next, often with inline dates
and people.

```
- [Recurring tasks](<feature record URL>): usability sessions showed people expect a skipped occurrence to stay skipped when the series is edited, and today it comes back. Sam Okafor and I agreed to fix that before the beta rather than after. It costs about a week, and I've told the two beta customers who were waiting.
- [Timesheet export](<feature record URL>): Globex asked whether the export can split hours by client as well as by project. It can't today; I've logged it as a request and will weigh it against approvals work at next week's planning.
```

Use this when each feature carries enough nuance to warrant its own
paragraph and the PM works across a single product area.

### "Additional work" sub-section

Both patterns commonly end with:

```
- Additional work
    - <small item 1>
    - <small item 2>
```

Anything that doesn't map to a tracked feature (small integrations,
customer-specific commitments, ad-hoc requests) goes here. Only add this
section if there's substantive non-feature work to report.

## Content coverage

Across the body (regardless of pattern), each feature bullet should touch
on whichever of these apply this week:

- **Progress**: what shipped, what advanced, customer interactions that happened
- **Decisions**: scope changes, target date changes, customer/stakeholder alignment moments
- **Blockers**: what's stuck and why, what would unblock it
- **What's next**: the next concrete milestone or date

Empty weeks: if a feature had nothing substantive happen, **omit it**. Do
not write "no update this week." Customer feedback is substantive: a
feature whose only activity this week is customer feedback gets one line
on what customers said. Keep the features-covered list (or relation)
tight to the body.

## Voice

Mirror the PM's voice from their prior updates. Style cues to copy:

- First-person or third-person, whichever they use
- Verb tense (most PMs use simple past for completed work and present/future for upcoming)
- How they write dates and refer to people (in tracker mode, inline date and person mentions if their prior updates use them)
- Abbreviation conventions (for example "BE" for back-end engineering, short customer nicknames)
- Comma habits, sentence length, formality

If the PM has fewer than 2 prior updates, fall back to
`config.style.fallback_style` (default: hybrid). Add a one-line note to the
draft handoff explaining the fallback.

**Hybrid** (the one definition, used by setup and the fallback): a bullet
per feature, 1 to 3 sentences each, plain past/present tense.

## Length

A typical update has 4 to 10 feature bullets of 1 to 3 sentences each,
and a body of about 150 to 500 words. When the material would run past 500
words, compress it instead of writing multi-paragraph bullets. Under 100
words is fine; short weeks happen.

## Drafting checklist before delivering

- [ ] Title matches `<PM name> - PM update - <YYYY-MM-DD>` exactly
- [ ] Every feature mentioned in the body is in the features-covered line (or tracker relation)
- [ ] No feature is in that list but absent from the body
- [ ] Structural pattern matches this PM's prior style (Pattern A or B), or the configured fallback when there are fewer than 2 prior updates
- [ ] Empty-coverage features omitted from the body; a feature with only customer feedback has one feedback line
- [ ] Every bullet is 1 to 3 sentences
- [ ] A feature whose record shows empty milestone or metrics fields says "no metrics set"; unknown (paste-only) says nothing; no invented metric commentary
- [ ] No filler bullets such as "no update this week"
- [ ] The body is 150 to 500 words, or shorter in a quiet week
- [ ] Dates written the way this PM writes them (tracker mode: date mentions if the tracker supports them)
- [ ] Nothing invented: every claim traces to a source read this run

If an item fails, fix the draft and run the checklist once more. If an item still fails after that second pass, deliver the draft and name the failing item in chat; don't loop.
