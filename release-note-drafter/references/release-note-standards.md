# Release Note Standards Reference

This reference contains the stable structure, format, and language patterns for customer-facing release notes. It's bundled with the release-note-drafter skill. For detailed style, voice, tone, terminology, and formatting rules, defer to the user's own style files (product style guide, UX writing guide, terminology glossary, punctuation standards) when they exist. Product-specific values (your product tags, your page's status labels, your official terms, hierarchy words, and extra banned phrases) are yours to set in the skill's config (see Setup in `SKILL.md`). The examples in this reference use a fictional work-tracking product (Timesheets, Calendar Sync, recurring tasks, Plan & Track); replace them with your own when you have them. A brand voice found by the voice lookup (`references/voice.md`) or in those style files shapes tone and word choice only: it never overrides this standard's structure, length, punctuation (no em dashes, en dashes, or ampersands), durability (no dates), or the mechanical check.

## What a release note is

A release note is a short, customer-facing announcement of a change a user will notice: a new feature, an enhancement, a rename, a change to a calculation or metric, or an availability change (open beta to GA).

A release note is not a full article. It announces what changed and why it matters, then links to the full article for how it works.

## Where release notes appear

The default model, which you can change: every entry goes on one release notes page that covers all of your products, and each entry shows there as a card. The page, not the entry text, supplies each card's status (open beta or GA, or your own labels for them), its product, and when it shipped, often as month groups. Readers see the title and roughly the first sentence or two before they open a card, and they're skimming cards from every product at once. So the title and the first sentence do most of the work, and the entry never repeats the status, product, or month.

If your page has no tags or uses per-product pages, keep the rest of this standard and report whatever metadata your page does use.

## What gets a release note (and what doesn't)

Release notes cover changes a user can perceive: a new feature or capability, a change to existing behavior, a change to a calculation, metric, or how data is reported, a rename, or an availability change (open beta to GA).

Do not write a release note for bug fixes, defect corrections, or backend or infrastructure changes with no user-facing effect. If the source is a bug fix or a non-user-facing change, say so and confirm with the user before drafting.

## Ground the entry in real context first

Never draft from the source description alone. Before writing, pull the real context for the feature so the entry is accurate and uses the correct terminology:

- Search connected docs (or ask the user) for the feature brief, conviction doc, spec, or product marketing material about this change. Use it to confirm the real feature and UI names, the availability status and tier, and the actual user value.
- Cross-check the user's terminology glossary, if there is one, for the product model and terminology traps that apply to this area.
- Confirm any product, feature, API, or SDK name (and its capitalization) against this context. Do not guess a name or its casing. If you can't confirm it, use a placeholder and flag it.

If the stated value doesn't logically follow from what the feature actually does, don't write a plausible-sounding benefit. Flag it for SME review instead.

## Availability: publish only open beta or GA

An entry can publish once customers can use the change, in open beta or generally available (GA). A closed beta isn't ready to publish yet.

- Closed beta: draft the entry anyway, flag it clearly, and advise the user to hold it until open beta or GA.
- Open beta: publishable. Say "available in open beta" plainly, without implying general availability.
- GA: publishable. When customers were already using a beta, say "now generally available (GA)". Spell out "generally available (GA)" the first time; "GA" alone is fine after that.

The status tag on the card shows open beta or GA, so don't restate the status in the body. The one exception is a beta-to-GA graduation, where the title or body can say the feature is "now generally available (GA)" because the change itself is the news. Spell out "generally available (GA)" on first use.

**Default to generally available.** If nothing in the source, docs, tickets, or code indicates a beta, a paid tier, or a gate, treat the change as shipped and generally available. Don't manufacture a caveat, and don't block on confirming universal availability. The one thing worth actively checking is a beta signal, since open versus closed beta decides whether the entry can publish at all.

## State availability and requirements only when they're a real gate

A release note announces what changed and links to the article for how it works. Don't add availability or requirement language by default. Include it only when the reader would otherwise wrongly assume they have the change, and keep it to one short, positively framed clause:

- **Paid add-on or tier** when the feature could be assumed available but isn't, and the product tag doesn't already make that clear. For example, timesheet approval rules that need the Business plan still carry the Timesheets tag, so note "available on the Business plan." Skip it when the product tag already names the paid product.
- **A setting, only when it gates access.** If a feature is off by default and won't appear until an admin turns on a setting, note that briefly as availability, such as "available after an admin turns on the Calendar Sync setting." Do not describe settings that are part of operating the feature; that's how-it-works and belongs in the article.
- **A minimum app or SDK version** when using the change depends on it, such as "requires version 3.0 or later of the desktop app."
If a source explicitly indicates a gate but doesn't give the detail, flag it. Otherwise, say nothing about availability and let the entry read as generally available.

## Entry structure

Each entry is exactly two parts, and both are required:

1. **Heading:** short, sentence case, no ending punctuation. Bold, not a markdown heading level, when placed inside a month section. Action-based is the default and fits most feature and capability additions. A status-style heading is appropriate for availability changes and renames, where it reads more naturally. Use judgment.
   - Action-based (default): `Pause a recurring task`, `Split a time entry`
   - Status-style (availability changes, renames): `Calendar Sync now generally available`, `Time logs are now Timesheets`
   - Avoid: `Big update!`, `Timesheet improvements.`, `We built task pausing`

2. **Body:** a single paragraph. No longer. If the update is complex, link to a full article rather than expanding the entry. The first sentence doubles as the card preview (see below).

## Title and first sentence: don't repeat

The title names the change. The first sentence adds what the title can't hold. This follows the same principle as modal and dialog copy: the title says what's happening, and the body gives the reader something new, such as the details, scope, or consequence. It never restates the title in other words.

The first sentence should add at least one of these:

- **Scope:** which objects, surfaces, or platforms it covers ("for a task and its subtasks," "in the desktop and mobile apps").
- **Specifics:** what the user controls or chooses ("for the number of weeks you pick," "by client, project, or person").
- **Behavior:** what happens as a result ("copies that already exist stay on the board").
- **Outcome:** what the reader can now answer or do that they couldn't before.

Run two tests:

- **Delete the first sentence.** If the entry loses no information, the sentence is repeating the title. Rewrite it.
- **Cover the title.** The first sentence should still make sense on its own, since some readers skim previews.

Sharing a noun with the title is fine ("recurring task" can appear in both). Restating the title's claim is not, including with synonyms or by adding "You can now" in front of it.

| Title | Repeats the title | Adds to the title |
|---|---|---|
| Pause a recurring task | You can now pause recurring tasks. | Skip upcoming copies of a recurring task for the number of weeks you pick, and the schedule picks up again on its own. |
| Split a time entry | Time entries can now be split. | Divide one block of logged time across two or more projects, and each part keeps the original notes. |
| Set a default project for Calendar Sync | Calendar Sync now supports a default project. | Meetings that don't match any project rule land in the project you choose instead of staying unassigned. |

**Keep the preview useful.** Put the most important new information in the first sentence, and keep it to about 25 words so it fits the card preview without being cut off mid-thought. Save the value statement for the second or third sentence.

## Required body format

The body follows this pattern:

1. Open with the most important detail the title doesn't carry, per the rule above. There's no fixed opening formula. Common openings that work: an imperative verb that adds scope or specifics ("Divide one block of logged time…"), a subject-first sentence ("Meetings that don't match any project rule now land…"), or "You can now…" when what follows adds to the title. Vary openings across entries, since they sit side by side on one page. Never open with "We've added…" or "We're excited to…".
2. State the value using an approved value pattern (see "Value statements" below). It can be its own sentence or folded into an earlier one with "so you can…" or "without…".
3. (Optional) Add context with **"Previously, you could only…"** to clarify what changed.
4. (Only if it's a real gate) State a requirement in one short clause, per the availability rules above: a tier the product tag doesn't show, an access-gating setting, or a minimum version. Never restate the status or product, since the card tags carry them.
5. Close with a link: **"For more information, see [Article title]."** If the article doesn't exist yet, insert a placeholder for the title and flag that the article is needed. Don't silently drop the link.

### Example (well-formed)

**Split a time entry**
Divide one block of logged time across two or more projects, and each part keeps the original notes. The parts appear in Timesheets as separate rows you can edit. This helps you bill each client correctly when one meeting covered work for more than one of them. For more information, see [Edit time entries](#).

The first sentence adds what the title doesn't (more than two projects, what happens to the notes). There's no availability line, because the product tag already tells the reader which product this is.

### Before and after (applying the rules)

This shows the recurring problems corrected in one entry.

This example uses the fictional work-tracking product. The Before deliberately contains an em dash, since removing it is one of the fixes; never copy it into an entry.

Before:
> We've added the ability to export timesheets by client — perfect for quickly preparing invoices! Admins can enable client exports from the Timesheet settings page as of this release.

After:
> **Export timesheets by client**
> Download a CSV of logged hours grouped by client, with one row for each person and project in the date range you choose. This helps you prepare invoices without copying hours out of each project. Available after an admin turns on client exports in Timesheet settings. For more information, see [Export timesheets](#).

What changed and why:
- Removed the em dash and the exclamation point.
- Dropped "We've added the ability to," which is announcement framing; the entry opens with what the user does.
- Cut "perfect for quickly," which is puff plus an ease claim; the value statement names the task instead.
- "enable" became "turns on."
- Cut "as of this release," which dates the entry.
- Kept the admin setting as one availability clause, because it gates access.
- Added the heading and the "For more information" link.
- The first sentence adds what the export contains, instead of restating "export timesheets by client."

## Value statements

Entries sit side by side on one page, so a single fixed value phrase repeats. Vary the phrasing, but only within these patterns:

| Pattern | Example |
|---|---|
| This helps you… | This helps you bill each client for the hours logged against their projects. |
| This allows you to… | This allows you to hold a weekly review over a holiday without deleting the series. |
| …so you can… (folded into another sentence) | …adds your meetings to Timesheets, so you can check the week before you submit it. |
| …without [the old cost] | Move a task to another person without reopening the project plan. |
| Previously, you had to… | Previously, you had to delete a time entry and log each part again. |
| Use it to… | Use it to find recurring tasks that have no owner. |

Don't use the same pattern in back-to-back entries. Every value statement must pass three tests:

- **Concrete.** It names a task, question, or cost the reader has. Abstract nouns such as "efficiency," "insights," "visibility," "productivity," or "your workflow" fail unless they're tied to a specific object ("find recurring tasks that have no owner" passes; "gain visibility" doesn't).
- **Sourced.** It traces to the problem statement in the conviction doc, spec, or brief. Name the source in the notes. If nothing supports it, use a placeholder instead.
- **Help-article voice.** It would read naturally in a help article. If it sounds like a landing page, rewrite it.

## Banned phrases

The mechanical check flags these automatically. Rewrite any hit unless it's part of a verified product or feature name on the check's allow-list (`release_note_drafter.glossary.allowed_names` in the skill's config, or a name verified this session).

- **Marketing verbs and adjectives:** unlock, empower, supercharge, elevate, streamline, seamless, effortless, game-changing, next level, powerful, robust, revolutionary, cutting-edge.
- **Your own banned phrases:** any terms in your saved glossary's banned list (for example, a positioning phrase your company uses in marketing that shouldn't appear in release notes). The check adds them to the list above.
- **Ease and speed claims:** quick, quickly, easy, easily, simple, simply, just. Describe what the user does instead.
- **Announcement framing:** "We're excited…", "We've added…", "Introducing…".
- **Punctuation that signals hype:** exclamation points and rhetorical questions.

## Terminology precision (recurring traps)

These mistakes recur in generated release notes. The rules are general; the examples use a fictional work-tracking product and are yours to replace. Your glossary supplies the actual values: official product and feature names, terms to avoid, and the hierarchy words your product uses (saved in the skill's config if you set them, or in your terminology and style files). Defer to those for the complete set; this is the high-frequency shortlist. Verify against the user's glossary and product docs when in doubt.

- **Give each thing one name.** Use the product's name only for the product, and its own words for the things inside it (individual "tasks", even though the feature is "Recurring tasks"). Don't fuse a product and one of its objects into a phrase that sounds like a feature: write "When a manager approves a timesheet…", not "Timesheets Approvals lets…". Use the official term rather than one you made up ("Timesheets", not "time tracker"), and don't attach a qualifier the concept's name doesn't have ("time entry", not "logged time entry", unless the extra word means something).
- **Pin down words that mean several things.** A word like "schedule" can mean a recurring task's repeat pattern, a person's working hours, or a project timeline, so name the one you mean. The same goes for words that name a level in your product's structure (list them in the glossary config): use one only when you mean that level, and otherwise say "people" or "your users".
- **Prefer a concrete name to an umbrella word.** Skip "insights," "solutions," or "flows" when there's a real name for the report or object (the timesheet export, the project timeline). Don't list one thing twice under two names (billable hours are a column in the timesheet export, not a second feature).
- **Say "turn on" and "turn off,"** not "enabled" and "disabled," or describe what the user does.
- **Verify API and SDK names and capitalization.** Don't invent or guess something like an SDK method or API surface name. Confirm it, or use a placeholder and flag it.

## Language patterns

- Openings that add to the title: an imperative verb with scope, a subject-first sentence, or "You can now…" followed by new detail. Vary them. Never "We've added…".
- Benefit statements: one of the approved value patterns, varied across entries (see "Value statements").
- Prior-state context: "Previously, you could only…"
- Status and product are carried by the card tags, not the body. Mention "now generally available (GA)" only for a beta-to-GA graduation, spelled out on first use.
- Requirement noted positively: "available on the Business plan," not "not available without the Business plan."
- Neutral, informative tone. No marketing language such as "powerful," "seamless," "modern," or "effortless."
- Present tense, second person, active voice.
- Describe outcomes and capabilities, not implementation detail.

## Durability rules

Write entries so they stay accurate after the release date and after a feature graduates from beta to GA. The page groups entries by month, so the body should not re-date itself.

Avoid time-bound or release-relative phrasing:

- Avoid: "In this open beta release…", "As of this update…", "New in June…", "We've just launched…", "Currently in beta…" (with no plan to update).
- Instead: state the capability plainly and let the status tag carry beta or GA.

When a feature graduates, the status tag is the only thing that should need updating.

## Punctuation

- **Never use em dashes.** This is the most common failure in generated release notes. Rewrite the sentence with a colon, a comma, parentheses, or a period. Run a final sweep for em dashes before presenting.
- Don't use en dashes for ranges. Use "to" instead, such as "5 to 10 minutes."
- **Never use ampersands.** Write "and". The only exception is an official product or feature name that contains "&", verified and on the check's allow-list.
- Use the Oxford comma.
- **Run a mechanical check before presenting.** Scan the entry text literally for —, –, and &, plus the banned phrases, exclamation points, and question marks, and rewrite every hit. Also scan for month names, years, and numeric date patterns (the durability rules above), and for ticket keys such as ABC-123 outside a link's URL. Length outside 50 to 90 words, not counting placeholder text, is a warning to note, not something to pad. Ignore only the article URL hidden inside the hyperlink markup (`[title](url)`), which can contain "&" in a query string. Also skip `[PLACEHOLDER: …]` spans, whose suggested SME question contains a question mark; check the text that replaces them once it's filled in. Count the skipped spans and report them separately as "Not ready to publish: N placeholders"; the count is not a check failure. A visible, bare URL is itself a failure: link the article title instead. Don't rely on reading it back.
- Follow the user's punctuation and numbers standards for everything else.

## Placement and ordering

- The page groups entries by month and sets the status and product tags. Report the month, status, and product alongside the draft as card metadata, never inside the entry text.
- Within a month, entries follow actual release order, most recent first. Don't reorder by perceived importance.
- Keep one capability per entry. Split unrelated changes into separate entries.

## Neutrality and value framing

- Convey the benefit factually: "provides automated insights," "makes reporting more consistent," "reduces manual input."
- Describe what's available and what the user can do. Don't lead with limitations or "unlike X" comparisons.
- If the value doesn't logically follow from what the feature does, flag it rather than writing a benefit that sounds right but isn't supported.
- No subjective or persuasive language. The goal is to inform, not to sell.

## Quick checklist

Run through this before finalizing any entry:

- Context pulled from connected docs or the user, and the glossary if present; feature, UI, and API names verified (or flagged)
- The change is user-facing (a UX or calculation change), not a bug fix or backend-only change
- Treated as generally available unless the source indicates a beta or gate (closed beta is drafted but held from publication)
- Status, product, and month reported as card metadata, not written into the entry
- A requirement is stated only when it's a real gate the tags don't show, framed positively; "generally available (GA)" spelled out on first use for a graduation
- Heading present: short, sentence case, no ending punctuation; action-based by default, status-style OK for availability changes and renames
- Body is a single paragraph
- First sentence adds information the title doesn't (passes the delete-the-sentence test), stays around 25 words, and works as the card preview
- Value uses an approved pattern, isn't the same pattern as the previous entry, and passes the concrete, sourced, and help-article tests
- No marketing language or subjective adjectives
- Mechanical check passed: no em dashes, en dashes, ampersands, bare URLs, banned phrases (including "Introducing"), month names, dates, ticket ids outside a link's URL, exclamation points, or question marks in the visible entry text (placeholder spans and allow-listed names excluded); length noted if outside 50 to 90 words, not counting placeholder text; "Not ready to publish: N placeholders" reported if any remain; ranges use "to"; Oxford comma used; if hits remain after two fix passes, the entry is handed back as not passed with each hit named
- Terminology and capitalization match the user's rules and the traps list above
- Links use the article title as link text, never "click here"
- Ends with "For more information, see [Article title]." (placeholder + flag if the article doesn't exist yet)
