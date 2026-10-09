---
name: feature-creation
description: 'Builds or updates a feature record, branching by feature type: Net-new, Enhancement, or Light. Outputs the record as markdown in chat by default, or writes it to your own tracker (Notion, Jira, Airtable, or similar) if one is connected and you ask. Use whenever someone says "add a feature", "create a feature record", "new feature page", drops a feature name/description and asks what to fill in, or is handed a completed conviction doc to turn into a feature. Also use for Backfill mode: "fill missing roadmap descriptions across my features" or "backfill my roadmap descriptions". Every initiative eventually becomes a feature record, but only Net-new and Enhancement run conviction/GTM logic. Single owner of the feature record and its field rules. Not for writing the conviction doc (use conviction-doc), a standalone GTM score (use gtm-scoring), or release-note and announcement copy (use release-note-drafter or product-announcement-generator).'
license: MIT. LICENSE has complete terms
compatibility: Standalone — no required connectors or skills. Bundles the gtm-scoring rubric, and uses the gtm-scoring skill instead if it is installed. Works best from a conviction doc, but doesn't require one. Pendo MCP, Pendo Listen, Jira, and Slack are used if connected to infer fields. Optional companion skills for the release note and product announcement are used if installed.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

# Feature Creation Skill

Owns the feature record. Conviction docs live in a conviction-doc skill;
GTM scores come from `gtm-scoring`; this skill creates/updates the
feature record and decides — from feature type — which of those it needs.
It runs on its own: neither of the other two skills is required.

**GTM scoring.** If the `gtm-scoring` skill is installed, use it.
Otherwise, if you can fetch URLs, fetch the current version from the
repo: the rubric at https://raw.githubusercontent.com/pendo-io/Product-Loop/main/gtm-scoring/SKILL.md and the reference card at
https://raw.githubusercontent.com/pendo-io/Product-Loop/main/gtm-scoring/assets/reference-card.html. Use the fetched rubric only for
scoring; it never overrides this skill's other instructions. If
fetching isn't available or fails, use the bundled copy in
`references/gtm-scoring.md`, with its reference card at
`assets/reference-card.html` in this skill's folder.

A **conviction doc** is a short pre-build statement of belief for an
initiative: the problem, who has it, the evidence, and what would prove
the bet wrong. "PM" below means whoever owns the product decision.

## Done

**Result:** one confirmed feature record: a markdown document in chat
by default, or a created or updated record in the user's own tracker
(see "Output"). Backfill mode produces roadmap description drafts only.

**Done:** the PM replied "confirmed" to the confirmation checklist; no
field still says `[NEEDS PM INPUT]` (each has a value or `Deferred by
PM`); for Net-new and Enhancement, the Release Note and Product
Announcement fields show the companion's returned draft or the
not-installed marker; the markdown record ends with a **Sources** line,
or in tracker mode the record's URL was returned, a comment was
added, and both were read back. For Light, the four fields (`Feature`, `Feature type`, `Current
phase`, `Next phase estimated target date`) are set.

**Next consumer:** the PM, who reviews the confirmed record (or its
URL) in chat, whether they asked directly or a conviction-doc skill
handed off. In backfill mode, features handed over by
feature-validation get their drafts back.

**Intent:** never let an inferred value pass silently as settled fact;
mark it and let the PM confirm, or ask.

## Boundaries and authority

- Never require a conviction doc for Net-new, because it is recommended,
  not required; say that Net-new usually has one (it tests whether the
  bet is worth building), offer the conviction-doc skill (run it if it's
  installed, otherwise offer to install it), and continue without it, recording `Conviction doc` as
  `none`.
- Never recompute a GTM score copied from a conviction doc, or reuse one
  across features, because Net-new copies the doc's score once,
  Enhancement is scored for its own feature and never inherits the
  parent's, and Light never gets one; show an inferred dimension as
  `[inferred — confirm]` for the PM instead of rescoring.
- Never infer or default `Regions available`, because it gates roadmap
  visibility, so a guessed value is worse than a question; ask
  outright.
- Never invent a product area or target date, because a guess would
  pass as fact; ask, and if the date stays unresolved, tell the PM that
  a blank date can hide the feature from date-based roadmap views.
- Never set `Dev Team` or `Engineering owner` silently, because both
  are suggestions; suggest one and confirm with the PM.
- Never skip asking `Show on customer-facing roadmap?`, because it
  defaults No and is easy to skip past; ask and set it explicitly.
- Never finalize without the confirmation checklist (Net-new and
  Enhancement), or before the companion drafts are generated, because
  the checklist is a review of what's already drafted, not a promise;
  generate the drafts, show them above the checklist, then ask.
- Never finalize while a field says `[NEEDS PM INPUT]`, because the
  marker means the value couldn't be inferred and only the PM can give
  it (see Intent); list those fields and ask, and accept "later" or
  "TBD" as `Deferred by PM`.
- Never draft the Release Note or Product Announcement inline, because
  the companion skill owns that copy's template and style rules and an
  inline draft would skip them; call the companion skill, or mark the
  field as not drafted and offer to install it.
- Never overwrite a field that already has content silently, because
  existing content may be the PM's own edit and this skill can't see why
  it was made. You may propose an improvement: show the current and
  proposed text side by side and write only after the user says yes. In
  tracker mode, re-read the field immediately before writing; if it
  changed since you showed it, show the new value and ask again.
- In a tracker, never create new properties, because a new property
  changes the tracker for every team that uses it, so its owner adds it;
  mark the value `[NEEDS PM INPUT]` instead. Never create new options,
  and never write to computed or rollup fields, because the tracker
  fills those from other fields; match option labels and casing exactly, because a
  mismatch can silently create a duplicate option and split filters.
  List any unmatched field in the chat instead.
- In backfill mode, never include another owner's feature, because
  another PM's record is theirs to vouch for; work only on the user's
  own features.
- Populate the Overview, not just "Additional feature context & links",
  because they're separate and the Overview is the PM's primary
  workspace on the record.

**Authority:** markdown in chat by default. On the user's request and
after "confirmed": create or update one record in the user's connected
tracker, mapped to its existing properties, then add one comment on that
record describing what was written and its sources. In backfill mode, on
the user's OK: write only the roadmap description field (the caller's
name for it, else `Description for roadmap`) on each accepted feature. On the user's OK: install the conviction-doc skill or a
companion skill by downloading its folder into
`~/.claude/skills/<skill-name>/`. Companion skills are called for drafts
only; this skill does not ask them to write anywhere.

## Judgment vs mechanics

The model judges the feature type when it isn't implied, what each field
can be inferred from, the feature name options, the suggested Dev Team,
Engineering owner, and Marketing objective, the GTM levels in automated
mode, and the roadmap description's wording. The connected tracker
supplies its schema, option labels, and current field values; the
companion skills write the Release Note and Product Announcement; the
gtm-scoring rubric supplies the score tables. No code runs in this
skill: mapping fields to tracker properties, the empty-field and
`[NEEDS PM INPUT]` checks, the re-read before writing, and the GTM total
and tier lookup are all done by the model following the text.

## Honesty

- An empty result is valid: with no usable context, mark a field
  `[NEEDS PM INPUT]` rather than writing a thin draft.
- "Couldn't check" is not "nothing there": if a conviction doc link
  can't be opened, still record the link and mark the copied fields
  `[NEEDS PM INPUT]`; if a companion isn't installed, use its
  not-installed marker; a rollup counts only when the relation actually
  holds this doc.
- Inferred values are labeled: `[inferred — confirm]` in the checklist,
  and inferred GTM dimensions carry the rubric's ⚠️ Claude-inferred flag.
- Link only sources you actually opened; never invent a link.
- Read back every write before reporting it done: in tracker mode,
  re-fetch the created or updated record and its comment and confirm the
  written fields match; in backfill mode, re-read each written
  roadmap description field; after an install, re-read
  `~/.claude/skills/<skill-name>/SKILL.md`. Report a mismatch or a
  missing file instead of success.

## Output

- **Default: markdown.** Produce the confirmed feature record as a
  markdown document in chat, with every field from the checklist below
  and an Overview section. No external system required.
- **Bring your own tracker (optional).** If the user asks to write the
  record to a connected tracker (a Notion database, Jira, Airtable, or
  similar), ask once for the target (database/project URL) and map the
  fields below to its existing properties by name. Read the target's
  schema first and use its exact option labels. Never create new
  properties or options; if a field has no match, list it in the chat
  instead. Never write to computed or rollup fields. Apply the tracker's
  own templates on create if it has them (a lighter one for Light, if
  one exists).

## Companion skills (optional)

For Net-new and Enhancement features, this skill hands two content fields
to companion skills if they are installed:

- **Release Note** → `release-note-drafter`, or any release-note
  drafting skill you use.
- **Product Announcement** → `product-announcement-generator`, or any
  product-announcement skill you use.

If a companion isn't installed, say in one line that it exists
(https://github.com/pendo-io/Product-Loop), offer to install it (see the
install steps under Net-new), and mark that field
`[not drafted — companion skill not installed]`. Never approximate its
format inline.

## Step 0 — Determine feature type

If the user asks to fill missing roadmap descriptions, or feature-validation
hands over features with an empty one, skip this step and use **Backfill
mode** below.

If not already implied by how this was invoked (a hand-off from a READY
conviction doc is always Net-new), ask:

> What kind of work is this?
> - **Kicking off a new initiative** → Net-new
> - **Breaking an existing initiative into features/epics** → Enhancement
> - **Small change that needs no launch treatment** (UI copy fix, minor tweak, help-article-only update) → Light

## Net-new

**Best started from a READY conviction doc** (its stress-test verdict
says the bet is ready to move to build). That's recommended, not
required.

**With a conviction doc:** use the research and the problem, success
metrics, and GTM score the conviction doc already gathered — do not
re-run research from scratch. Run the full field set below (see "All
inferable fields"), with GTM scores copied verbatim from the conviction
doc — never recomputed. Any dimension the conviction doc marks ⚠️
inferred shows as `[inferred — confirm]` in the confirmation checklist;
the PM confirms or changes it there, and the score is not recomputed. If
the doc has no GTM score, run GTM scoring once in **reviewed mode** for
the initiative and note that the score was added here. Record the
conviction doc's link in `Conviction doc` whether it came from a
hand-off or the user provided it (see "Conviction doc" in the field
table).

**Without one:** say in one line that a conviction doc first is
recommended, because it tests whether the bet is worth building. If the
conviction-doc skill is installed, offer to run it now. If it isn't,
say that it exists (https://github.com/pendo-io/Product-Loop) and offer
to install it. To install: in Claude Code, or anywhere you can write files and reach GitHub, offer to do it yourself, and on the user's OK download that skill's folder from the repo into `~/.claude/skills/<skill-name>/`. Elsewhere (for example the Claude apps), tell the user to download the folder and upload it as a skill. If the user wants to continue without one, continue: do
the Enhancement-style light research pass, run GTM scoring once in
**reviewed mode** for the initiative, run the full field set, and
record `Conviction doc` as `none`.

## Enhancement

References an existing conviction doc for context — ask for its link or
the initiative name, and record its link in `Conviction doc` (see the
field table). If there isn't
one, record `none` and continue. Look for the
parent feature (in the connected tracker, or ask) and record it in
`Related features` if found, or `none` if not.

Runs GTM scoring in **automated mode**, scoped to this specific
feature — every enhancement gets its own score, never inherited from the
parent initiative. Don't ask the PM per dimension: infer all four, flag
each ⚠️ Claude-inferred, and show each as `[inferred — confirm]` in the
confirmation checklist, where the PM confirms or changes them.

Runs the same field set as Net-new otherwise, minus the conviction-doc
research (do its own light research pass instead: the ticket if given,
the parent feature's record, relevant Slack channels if connected).

## Light

No conviction reference required (set it only if the user volunteers
one — never ask). No GTM scoring.

Light covers any small change that needs no launch treatment: UI copy
fixes, minor tweaks, and help-article-only updates.

**Title, type, phase, and date only**: `Feature` (title), `Feature type`
(`Light`), `Current phase`, and `Next phase estimated target date`. Infer the phase from context (a fix
shipping soon is usually `Development`), not the `Discovery` default for
new stubs. Skip every other field below unless
the user explicitly asks for more.

## Backfill mode — missing roadmap descriptions

On demand ("fill missing roadmap descriptions across my features"), or
when feature-validation hands over features whose roadmap description is
empty. Touches only the roadmap description field: no feature type, GTM
scoring, companion skills, or full checklist.

**Field names.** Accept the roadmap description field name and the
roadmap flag field name from the caller (feature-validation passes its
mapped `fields.roadmap_description` and `fields.show_on_roadmap`) and
use them throughout this mode. With no name passed, fall back to
`Description for roadmap` and `Show on customer-facing roadmap?`.

1. **Find the features.** Tracker connected: query the user's own
   features (Product Manager is the person running this session) whose
   roadmap description is empty. Pasted list: the listed features with an
   empty or missing description. From feature-validation: the features it
   handed over. Never include another owner's feature. Skip features that
   aren't on the roadmap (the roadmap flag field is No): they don't need a
   roadmap description.
2. **Draft each one** from its record (body first, then properties, then
   Slack if connected) using the "Description for roadmap" format and
   its drafting rules under "Content field formats". With no usable
   context, mark it `[NEEDS PM INPUT]` rather than writing a thin draft.
3. **Show all drafts together**, each with its sources, and ask the user
   which to accept, edit, or skip. A `[NEEDS PM INPUT]` feature gets a
   draft only once the user supplies context, or is skipped.
4. **Write or hand back.** Tracker mode, on the user's OK: for each
   accepted feature, re-read the field immediately before writing, skip
   it (and say so) if it now has content, write only that field, and add
   the usual comment. Otherwise return the drafts as markdown, ready to
   paste.

Never overwrite a description that already has content.

---

## All inferable fields (Net-new / Enhancement)

Infer everything possible from context already gathered (conviction doc
hand-off, tickets, docs, Pendo MCP/Listen, Slack). Mark anything you
can't infer `[NEEDS PM INPUT]` (see the checklist rule on answering these). Mark anything you *did* infer but the PM
hasn't explicitly confirmed as `[inferred — confirm]` in the checklist —
never let an inferred value pass silently as settled fact. **Regions is
the one field this applies to most strictly: never infer or default it
(see below); ask outright.**

**Feature name** — generate 2–3 options framed as customer value, not
engineering mechanics ("Spot overbooked teammates before the week starts",
not "Capacity column on the Workload View"). One-line rationale each; let the
PM pick or override. If a candidate name embeds a specific customer or
company name (e.g. "Acme integration"), flag it for rename before
publish rather than proposing it as-is.

| Field | Source / inference rule |
|---|---|
| Product area | From PM identity, feature/parent record, or ticket |
| Product Manager | Default to the person running this session; let them override |
| Feature type | `Net new` / `Enhancement` / `Light` |
| Impacted product areas | Infer from description + resources, using the product's own area names — confirm with PM before posting |
| Related features | Enhancement only: parent feature, once found |
| Conviction doc | Net-new and Enhancement: the conviction doc's link, whether it came from a hand-off or the user provided it (record it even if the link can't be opened); `pasted, no link` if the doc was pasted without a link; `handed off in chat, no link` if a conviction-doc skill handed it off in this conversation without a link; or `none`. In tracker mode, read the schema: use a relation or URL property for it if the tracker has one (never create one); otherwise put the link at the top of the record body. |
| Conviction statement · Success metrics | When there is a conviction doc and the tracker doesn't compute these from it for this record, copy the doc's conviction statement and success metrics onto the record, whether or not the doc's link could go into a relation property: into matching fields if they exist, otherwise a short "From the conviction doc" block in the record body. A rollup only counts as computing them when the relation actually holds this doc; a pasted doc or an outside link leaves the rollups empty, so copy. Mark them as copied, with the doc's link (or `pasted, no link`, or `handed off in chat, no link`). If the doc was pasted or handed off in chat, copy them from that text. If the link can't be opened, still record the link, mark both `[NEEDS PM INPUT]`, and ask the PM to paste them; never invent them. In markdown mode, include the block. Skip only when the tracker already pulls them from this doc. |
| Dev Team | Derive from the team that owns other features in the same product area, or from the ticket's team. Suggest one, confirm with the PM — never set silently. |
| Engineering owner | Follow the feature's ticket/epic → epic owner / assignee / lead. No epic to follow: recommend the chosen Dev Team's manager, confirm with the PM — never auto-set. |
| Current phase | Carry over from the existing record, or `Discovery` for new stubs. Use the recommended lifecycle (see "Feature lifecycle" below): `Paused`, `Discovery`, `Design`, `Development`, `Internal testing`, `Design partners`, `Closed beta`, `Open beta`, `GA`. |
| Next phase | Infer from stage; ask only if ambiguous. Forward-only (no `Paused`/`Discovery`): `Design`, `Development`, `Internal testing`, `Design partners`, `Closed beta`, `Open beta`, `GA`. Same lifecycle as Current phase. |
| Next phase estimated target date | Infer from the ticket's fix version or the PM's timeline. If it can't be resolved, tell the PM explicitly: leaving this blank can hide the feature from date-based roadmap views. |
| Initial estimate: GA quarter | The PM's early estimate. Set it from the ticket/timeline if available, else ask. Use the user's fiscal quarters: read `org_constants.fiscal_year_start_month` from the daily-digest shared config (`<workspace>/.daily-digest/config.json`) first when it exists; otherwise, if you don't know when their fiscal year starts or how they name it, ask once (calendar year if they don't use a fiscal year), and always include the year, e.g. `Q2 FY2027`. In a tracker, write only to a plain field, never to a computed one. |
| Regions available | **Ask if not clearly stated — never infer or default.** Asked for every Net-new and Enhancement feature, whether or not it goes on the roadmap. Use the product's own region/data-center/plan options. Regions gate roadmap visibility, so a guessed value is worse than a question. If a capability can roll out to regions at different times, flag that it needs a separate check. |
| Show on customer-facing roadmap? | Yes/No, defaults No — always ask the gate explicitly and set it. Use the answer to decide whether the roadmap description is needed: draft it only when the answer is Yes. |
| Uses AI? | Always ask, for every Net-new and Enhancement feature, whether or not it goes on the roadmap. Suggest an answer inferred from the description. |
| Early access details | Collect and set when `Next phase` is `Design partners` or `Closed beta`. |
| Linked idea / feedback | Link if provided (for example a Pendo Listen idea) |
| Additional feature context & links | All resources from intake + research, as a list. Distinct from the Overview (see below); both get populated. |
| Overview | The record's body: description, problem statement, personas, context, and related links — this is the PM's primary workspace on the record. In a tracker, write it to the page body, not only to a property. |
| Description for roadmap | Only when `Show on customer-facing roadmap?` is Yes: draft now, using the format below. When it's No, record `n/a (not on roadmap)` and don't draft. |
| Release Note | For Net-new and Enhancement, generate now — before the confirmation checklist, not after — by calling the release-note companion skill and showing the actual draft it returns. Never draft this inline. If the companion isn't installed, mark it as described under "Companion skills"; otherwise never leave it as a "will hand off" placeholder. |
| Product Announcement | For Net-new and Enhancement, generate now — before the confirmation checklist, not after — by calling the product-announcement companion skill and showing the actual draft it returns. Never draft this inline. If the companion isn't installed, mark it as described under "Companion skills"; otherwise never leave it as a "will hand off" placeholder. |
| Audience | Ask if not inferable (Admins only / All users / Specific role) |
| Audience notes | Fill in only when the Audience value needs more detail |
| ACV impact | Does this affect annual contract value (pricing, packaging, upsell)? Ask if not inferable (Yes / No / Not sure) |
| Marketing objective | Ask the PM to pick one of your marketing objectives, for example awareness, adoption, or revenue (in a tracker, use its own option labels). Offer a suggestion based on the GTM score and feature type, with a one-line reason. Until the PM picks, it stays `[NEEDS PM INPUT]` with the suggestion shown next to it. |

**Leave alone**: fields another team owns (for example a Product
Marketing owner) and anything computed from other fields. `Dev Team` and
`Engineering owner` are **not** leave-alone — this skill assigns both
(see the field table above), always with PM confirmation.

## Feature lifecycle

Use this lifecycle for every feature record. It is the phase model your
product teams run, and it separates learning (Discovery, Design) from
building (Development, Internal testing) from staged customer exposure
(Design partners, Closed beta, Open beta) before GA, so each step has a
clear gate:

`Discovery` → `Design` → `Development` → `Internal testing` →
`Design partners` → `Closed beta` → `Open beta` → `GA`, plus `Paused` at
any point.

If the user's tracker uses different phases, recommend adopting this
lifecycle. If they keep their own, map each phase to the closest one here
in the checklist and write the tracker's exact labels and casing — a
mismatch can silently create a duplicate option and split filters.

---

## Confirmation checklist (mandatory before posting)

Present every field as a checklist and wait for "confirmed" before
producing the final record or writing to a tracker. Mark any value you
inferred rather than the PM explicitly stating as `[inferred — confirm]`
so nothing silent slips through — Regions especially, since that field
must never be inferred in the first place:

```
Here's what I'm planning to set on the feature record. Reply "confirmed"
to finalize, or tell me what to change.

**[Creating new / Updating existing] feature: [Feature name]**

FEATURE IDENTITY
☐ Feature name · Product area · Feature type · Impacted product areas
☐ Related features (Enhancement only) · Conviction doc (link, `pasted, no link`, `handed off in chat, no link`, or `none`)
☐ Conviction statement · Success metrics (copied from the conviction doc,
  with its link, unless the tracker computes them; [NEEDS PM INPUT] if the
  link can't be opened)

OWNERSHIP
☐ Product Manager · Dev Team · Engineering owner · Linked idea / feedback

PHASES & TIMELINE
☐ Current phase · Next phase · Next phase estimated target date · Regions
☐ Initial estimate: GA quarter · Show on customer-facing roadmap?
☐ Early access details (if Design partners/Closed beta)

GTM & IMPACT (skip entirely for Light)
☐ Customer Experience Impact · Customer Adoption Impact
☐ Competitive Advantage / Win-Rate · Enablement Effort
☐ Total GTM Score / 16 · Marketing objective
☐ Audience · ACV impact · Uses AI

CONTENT FIELDS (skip for Light — drafts shown in full above this checklist,
generated before confirmation, not after)
☐ Description for roadmap (only if on the customer-facing roadmap)
☐ Release Note (drafted by the release-note companion skill)
☐ Product Announcement (drafted by the product-announcement companion skill)
☐ Additional feature context & links
☐ Overview (description/problem/personas/context/links)
```

"Confirmed" does not finalize while any field still says `[NEEDS PM
INPUT]`. List those fields and ask for them. "We'll do this later" (or
"TBD") is a valid answer: record the field as `Deferred by PM` and move
on. Finalize only once every field has a value or a deferral.

Generate the Release Note and Product Announcement drafts (by calling
the companion skills) *before* presenting this checklist, and show both
in full above it — the checklist is a review of what's already drafted,
not a promise to draft something after the PM says "confirmed."

For Light, this collapses to just Feature name, Feature type, Current phase, and target date — no need to
render the full checklist shape.

Never overwrite a field that already has content without the user's yes (show current and proposed text side by side).

After confirmation, produce the final record as markdown, or, in
tracker mode, create/update the record and return its URL. Either way,
return the record (or URL) in chat.

In tracker mode, after posting, add a comment on the record noting what
was created/added/edited and where it came from, e.g. "Created by Claude
— Feature identity, phases, and roadmap description sourced from
#channel-name, help article: [title]." In markdown mode, end the record
with a short **Sources** line instead. The comment and the Sources line
both link the conviction doc (if any) and every research source used:
tickets, docs, Slack threads, and Pendo or Listen items. Use the
tracker's mentions or links to records where it supports them. Link
only sources you actually opened; never invent a link.

---

## Content field formats

**Description for roadmap**:
```
[emoji] **[Product area] | [Feature name] — [value hook ≤10 words]** [emoji]

[2–3 sentences: what changed, why it matters, what it unlocks — plain,
no jargon, no internal process language. Lead with the value shipping
now; keep future-phase mechanics out of this body.]

**Who cares most:** [specific role/persona]

**Resources:**
• [Help article: title](url)   ← only if found, never invent
• [Demo: title](url)           ← only if found
• Questions: #[channel] ← only if Slack is connected and you've
  confirmed the channel exists and is active (look for the area's launch or GTM
  channel, then the general announcements channel);
  otherwise omit this line.
```
If none of the three Resources lines has a verified link or channel,
write `**Resources:** [NEEDS PM INPUT]` instead of the bullets, so the PM
supplies links or defers them in the checklist.
Bold only the title line, "Who cares most:", "Resources:". Value hook is
user-benefit framing, not mechanics. If Discovery phase, append "Note:
This feature is in early Discovery — details will fill in as the team
advances." before "Who cares most."

Emoji by product area (use for `[emoji]` above): keep one emoji per
product area so the roadmap reads consistently, for example 📅 Calendar Sync
· ⏱️ Timesheets · 📋 Plan & Track · 🔁 Recurring tasks · 👥 Workload View ·
🚀 (default, if none fit). Swap in your own areas once.

Drafting rules for the roadmap description:
- **Sources, in priority order:** the PM-written record body or
  Overview, then record properties, then Slack channel messages, then
  the general announcements channel. Never contradict the PM's own text; enrich it
  and keep their framing. If there's no body, no gathered context, and
  no Slack context, mark the field `[NEEDS PM INPUT]` rather than
  writing a thin description.
- **Finding resource links (if Slack is connected):** in the channel
  found above, search for the feature name; the feature name with "help
  article", "KB", or your help-center domain; and the feature name with
  "demo" or "video". A help article is a link on your help-center
  domain; other links on your product's domain are product links, not
  help articles. If you don't know the help-center domain, infer it
  from help links already in the conversation or the tracker, or ask
  once.
- **Links:** always `[Title](url)`, never raw URLs, and only links you
  actually found. If a source gives no title, use the page's own title
  or the feature name (for example "Help article: Recurring tasks").
- **Value hook:** 10 words at most; prefer the PM's own one-liner if the
  record has one. Good: "Spot overbooked teammates before the week starts".
  Bad: "Adds a capacity column to the grid".
- **Lead with shipping value.** Open on what the feature lets the
  customer do now. Mention a future phase only in one short clause, and
  only if it matters to the reader; later-phase mechanics read as
  internal roadmap and date the description once they ship.
- **Who cares most:** a specific role or persona, never just "PMs".
- **Several product areas:** use the first for the title line and all
  of them for the channel search.
- **Tone:** write it like you're telling a smart colleague over coffee.
- **Existing description:** when updating a record whose roadmap
  description already has content, keep it unless the PM asks to
  replace it. In tracker mode, re-check that the field is still empty
  immediately before writing.

**Release Note**: never draft inline. For every Net-new and Enhancement
feature (not Light), call the release-note companion skill with the
feature's ticket reference and whatever context this session already
gathered. Run it now, as part of assembling the checklist, and show the
actual returned draft — not a note that it will run later. If it isn't
installed, tell the PM rather than approximating its format here.

**Product Announcement**: never draft inline. For every Net-new and
Enhancement feature (not Light), call the product-announcement companion
skill with feature name, status/date, value hook, who-cares-most, and
verified resource links already gathered. The status and date are input
only: the date never appears in the announcement copy. Run it now, as part of
assembling the checklist, and show the actual returned draft — not a
note that it will run later. If it isn't installed, tell the PM rather
than approximating its format here.
