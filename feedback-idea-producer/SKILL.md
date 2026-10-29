---
name: feedback-idea-producer
description: 'Produces a digest of what customers are saying: pulls recent Pendo Listen feedback for your focus areas (or feedback you paste or export), filters out system-ingested noise, clusters it into typed themes (bug / feature request / pain point / praise), spots what is NEW since the last run, matches top themes to backlog ideas and proposes feedback links for your OK (automatic only if you turn that on), and suggests connecting each matched idea to its feature record. Renders a digest in chat, or a structured update for a digest composer. Single owner of linking Listen feedback to backlog ideas and of the feedback lens in the daily digest. Not for support-ticket trends (use support-trends-producer), call signal (use field-qual-producer), or composing the digest (use digest-composer). Triggers: "what are customers saying", "triage my feedback", "feedback themes", "what should I prioritise", "cluster my feedback", "link feedback to ideas", "daily/weekly feedback digest". Can also run on a schedule.'
license: MIT. LICENSE has complete terms
compatibility: Standalone. Works on pasted or exported feedback with no connectors. Pendo MCP (Pendo Listen feedback and ideas) is the primary optional source and enables idea matching, linking, and ARR. Optional feature tracker (Notion, Jira, Airtable, or similar) for the idea-to-feature suggestion. Optional Slack for posting the digest. Feeds digest-composer if installed.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

## When to use

Use for a daily or weekly product review, feedback triage, or backlog grooming grounded in qualitative evidence. Triggers: "what are customers saying", "triage my feedback", "feedback themes", "what should I prioritise", "cluster my feedback", "link feedback to ideas", "daily/weekly feedback digest". Can also run on a schedule.

# Feedback & Idea Producer: the qualitative signal for the daily digest

A **producer**: it owns one source (Pendo Listen feedback, or feedback you provide) and produces a
short, structured, **delta-only** update each run. It works two ways:

- **On its own** (the default when nothing else is installed): it renders a readable feedback
  digest in chat (see "Standalone digest" below).
- **Alongside a digest composer** such as `digest-composer`, or an orchestrator that runs producers
  daily: it hands over the structured output in Step 6, and the composer places it under
  **💬 From the field & feedback** in the daily digest. Those skills are optional and live at
  https://github.com/pendo-io/Product-Loop. If they aren't installed, mention once that they exist
  and carry on.

It does **not** create ideas without approval. Its job: turn the window's raw feedback into
*"here's what customers are newly telling us, what type of thing it is, whether it's escalating,
and which backlog ideas it maps to"*, **linking** the new feedback to the matched idea (logged and
revertible), and **suggesting** the connection between that idea and its feature record.

"PM" below means whoever owns the product area. "Focus areas" are the product areas the user
cares about.

## Done

- **Result:** a readable feedback digest in chat (Step 7), or, under a composer, the structured
  object in Step 6.
- **Done when:** the digest has its header line; every surfaced theme carries a type, "N similar
  items · M accounts", and a real attributed quote; the Set aside, Coverage, and Idea backlog lines
  are present; every applied link is logged with its idea id and feedback ids; `last_run` is
  updated. A quiet run is the calm line with its direction instead of themes. Under a composer, the
  Step 6 object carries every key.
- **Next consumer:** `digest-composer` or the `daily-digest` conductor, which places it under
  💬 From the field & feedback. Run alone, the person who asked.
- **Intent:** discovery, not recap. When steps run out, name the gap rather than pad a theme.

## Boundaries and authority

Hard stops:

- Never create an idea on your own, because creating is the user's call (the autonomy boundary in
  Step 5); stage it as a 🟡 `create_idea` with the draft and call `createIdeaItem` only after
  approval.
- Never invent feedback, themes, quotes, or ids, because a quiet day is a valid answer and the
  composer merges on real ids; with nothing provided or connected, say so and stop, and with no ids
  emit an empty provenance set.
- Never take counts, quotes, or themes from `generateFeedbackTopics` or `getFeedbackInsights`,
  because they cannot be filtered by author and include system-ingested items; count from your own
  assignment of the kept `getFeedbackItems` items.
- Never infer an account from an email domain or a name. Account names aren't unique; resolve by
  id, or keep the item's own account and say so.
- Never guess `idea_field` or create a tracker property; read the tracker schema, and if the field
  can't be resolved, skip the feature hop and name the gap.
- Never prompt on a scheduled run, because scheduled runs are non-interactive; use config, apply
  links only when `auto_link` is true, and stage everything else as 🟡.

**Authority.** Every write this skill can make:

- Pendo Listen: `linkIdeaAndFeedback` on 2 to 3 representative unlinked items per confident match,
  after one confirmation, or without asking when `auto_link` is true. Each applied link is logged so
  it can be reverted with `unlinkIdeaFeedback`; the skill does not call the revert itself. Read
  each link back (the item's `linkedIdeas` from `getFeedbackItems`) before reporting it ✅.
- Pendo Listen: `createIdeaItem`, then `linkIdeaAndFeedback` on its representative items, only
  after the user approves. Re-read the new idea with `getIdeas` and its links as above before
  reporting it created.
- Feature tracker: sets `idea_field` on a record only after the user approves, then re-fetches
  the record to confirm the value.
- Slack or another channel: posts the digest only when `feedback_idea.standalone_digest` is set,
  and confirms it by opening the returned message permalink.
- Its own `config.json`: `last_run` every run, the `idea_index` cache at setup or refresh, and
  focus areas or other values when the user agrees to save them. Re-read the file after each write.

A write that doesn't read back is reported as unconfirmed, not done. Everything else is read-only.

## Judgment vs mechanics

The model judges: clustering kept items into themes, each theme's type, urgency in the escalation
scan, whether an idea match is confident, quote choice, and phrasing. Pendo MCP tools gather the
feedback, ideas, existing links, and account metadata. These mechanics are still done by the model
from prose and are candidates for code: sub-window merging and the per-day coverage tally, item and
distinct-account counts, distinct-source counting, the NEW-since-last-run mark, backlog slice
merging and the full-read count check, and the ARR sum and concentration share.

## Honesty

- An empty result is valid: emit the calm line with its direction (Step 6), never a padded theme.
- Couldn't check is not nothing there: a capped pull gets a coverage note, a partial backlog gets
  "couldn't check the full idea backlog (read N of M)", an unset tracker or `idea_field` is skipped
  and named, and no source at all means say so and stop.
- Label proxies and partial samples: "ARR resolved for N of M accounts" with any concentration
  flag, machine-written and imported items labeled wherever they appear.
- No time estimates.

**Before presenting**, check: no capped pull without its coverage line; no `-42` item counted when
its owning producer ran; no theme without a quote; no stub `idea_link`; no `one_line` that restates
its quote. Fix what fails and check once more; if a check still fails, ship with the failure named.

## Sources, in order of preference

1. **Pendo MCP connected** (Pendo Listen): the full workflow below, including idea matching,
   linking, and ARR.
2. **No Pendo MCP:** ask the user to paste feedback or share an export (CSV or similar; ideally
   with item id, date, author, account, text, and product area). Run Steps 1 to 4 and the
   rendering rules on that data. Idea matching (Step 5) runs only against an idea list the user
   also provides, and only as suggestions: nothing is linked. A theme with no match in that list
   and enough distinct sources still gets a new-idea draft, handed over as text; with no idea list
   there is nothing to check a match against, so skip drafts and say so. ARR (Step 5c) runs only if
   the export carries account ARR. Say which steps were skipped and why.
3. Nothing provided and nothing connected: say so plainly and stop. Never invent feedback.

## Config it reads (never hardcode identity)

All identity comes from config. When a composer or orchestrator is present it may supply these;
when run alone, keep them in a local `config.json` (shape in `config.example.json` in this folder)
or ask for them once and offer to save:

- `focus_areas`: the user's product area(s); resolve to Pendo **product-area IDs** (under the
  conductor, use its `pendo.product_area_ids` when set). If none set, fall back to the full
  firehose and say so.
- `feedback_idea.subscription_id`: the Pendo subscription that holds your Listen data (from the
  Pendo connection). When the daily-digest conductor runs this skill (or its shared config exists
  at `<workspace>/.daily-digest/config.json`), read the conductor's `pendo.subscription_id` first,
  then this key.
- `feedback_idea.window_days`: how many days back to look. Default 7; when the window holds fewer
  than about 15 items, widen it to 14 on your own.
- `feedback_idea.auto_link`: whether confident feedback-to-idea links are applied without asking
  (default `false`; see Step 5).
- `feedback_idea.standalone_digest`: optional `{channel, cadence}` if the user also wants the
  categorized digest posted somewhere (for example a Slack channel). Default off; the digest
  renders in chat.
- `feedback_idea.feature_tracker`: optional. The feature tracker (a Notion database, Jira
  project, Airtable base, or similar) to reconcile ideas against, as `{type, ref}`. If unset, skip
  the idea-to-feature hop and say so.
- `feedback_idea.idea_field`: the feature-record property that stores the linked **idea** (for
  example a Pendo Listen idea). Used to detect whether an idea is already connected to a feature and, if not, to suggest
  the connection. Read the tracker's schema to confirm it exists. Never create it. If it can't be
  resolved, skip the tracker hop and say so (don't guess a property).
- `feedback_idea.arr_field`: the account-metadata field that holds ARR in your Pendo
  subscription (for example `account.<your_crm_group>.<arr_field>`). No default: ask at setup.
  Used only for the fallback ARR computation (Step 5c).
- `feedback_idea.crm_account_id_field`: optional. The account-metadata field that holds the
  account's CRM (for example Salesforce) id, used to emit the `account:crm:` provenance key.
- `feedback_idea.new_idea_min_sources`: how many distinct sources an unmatched theme needs before
  the producer drafts a new idea for it (default `3`; see Step 5).
- `feedback_idea.arr_concentration_threshold`: flag single-account ARR concentration at/above
  this share (default `0.5`).
- `feedback_idea.idea_url_pattern`: URL template for hyperlinking an idea by id (for
  example `<your Pendo app URL>/s/{subscriptionId}/listen/validate?openItem={ideaId}`); render
  the idea **title** as the link text. Read `org_constants.feedback_idea_url_pattern` first when
  the daily-digest conductor passes it (or its shared config exists at
  `<workspace>/.daily-digest/config.json`), then this key. If neither is set, reference the idea
  by title without a link, plus its id in plain text when the id is known.
- `feedback_idea.idea_index`: the cached map of existing backlog ideas in the user's areas (idea id
  to title and area), built in Setup step 2 and refreshed weekly or on demand, so each run matches
  themes against a known set. With a pasted idea list, build it from the paste. Starts empty.
- `last_run`: timestamp of the previous run, for the NEW-since-last-run delta. If absent, treat
  everything in the window as new and say so.

Every run reads `last_run` and `idea_index` from config, never from the conversation.

On an interactive run the user may name a product area or window inline. On a **scheduled run
never prompt**: use config, apply links only if `auto_link` is true, and stage everything else.

## Setup (first run, then refresh occasionally)

Ground the skill in the person's world before the recurring loop runs.

1. **Confirm focus areas: the FULL scope, not a subset.** Read `focus_areas`; if empty, ask once and save. **If the user leads multiple areas** (has PMs reporting to them), resolve their *entire* portfolio (every product area they own) and treat all of them as in-scope. Do **not** quietly narrow to a convenient subset: pull each area in the sweep and account for **all** of them in the output, including the quiet ones (render a swept-but-empty area explicitly, for example *"Calendar Sync: quiet"*), so a leader can trust the read is complete. If you cannot resolve an area to a Pendo product-area ID, say so rather than dropping it silently. (With pasted data, match areas to the product-area labels in the data instead.)
2. **Figure out the relevant existing ideas.** For those areas, pull every current backlog idea
   (`getIdeas` scoped to the areas, read in full as in Step 5) and build a lightweight map of what already exists, so each
   run matches against a known set rather than searching cold. Cache it in
   `feedback_idea.idea_index` (refreshed weekly or on demand).
3. **Reconcile ideas and feature records** (only if a feature tracker is configured). For each of
   those existing ideas, check the `idea_field` on records in `feedback_idea.feature_tracker`.
   Present the connections that are **missing** as a batched 🟡 (*"these {N} ideas in your areas
   aren't linked to a feature record yet. Connect them?"*) so the person establishes the mapping
   once. After setup, each run only has to handle the *new* ideas that surface.

This is a setup/refresh activity, not part of every run; the workflow below assumes the index
exists (and rebuilds it if missing).

## Workflow

### 1. Pull raw feedback (delta-aware)
`listAllApplications` to confirm the subscription. Then `getFeedbackItems` with the subscription,
the `productAreaIds` resolved from `focus_areas`, `startDate` = window start, `endDate` = tomorrow
(`endDate` behaves as exclusive, so an end of today would drop today's items).
Mark each item **NEW** if its submission date is after `last_run` (the discovery delta); older
items are context, not surfaced unless part of an escalating trend. (With pasted data, use the
item dates the same way.)

**Result cap.** `getFeedbackItems` returns at most about 30 items per call and has no paging. If a
call comes back with 30 (or close to it), or reports more matches than it returned (for example
"most recent 30 of N"), assume it was truncated: split the window into smaller date ranges (or one
call per product area) and merge by item `id`. Because `endDate` is exclusive, make each
sub-window's `endDate` the next sub-window's `startDate` (for example `<date A>` to `<date B>`, then
`<date B>` to `<date C>`), so no day falls between two ranges. **Coverage check:** tally the merged items per
submission day and confirm the per-day counts across all sub-windows add up to the merged total
and, when the tool reports one, to its total for the full window; a day with no items inside a busy
window, or a merged total below the reported total, means a gap, so re-pull that day. Always record
a **coverage note**: how many items were retrieved against the reported total, over which ranges,
and whether any range still hit the cap (for example *"Coverage: N of N over 2 date ranges; no
range hit the cap"*, or *"Coverage: N items over 2 date ranges; 1 range returned 30, so it may be
incomplete"*). Render it in the standalone digest and append it as a final sentence of `insight` in
the structured output. Never present a capped pull as the complete window.

### 2. De-duplicate against the support and field producers
Items with `createdBy.id == "-42"` (Listen's `--system-user--`) are auto-ingested from integrations
such as support-ticket and call-recording tools, which is **the same underlying signal the support
and field producers** (`support-trends-producer` and `field-qual-producer`) own. What to do with them
depends on whether those producers run:

- **The matching producer runs this run** (for example a conductor or composer lists it among the
  producers it ran or will run): set those items aside from the human-submitted feedback this producer
  reports, and record the count **with the reason** in `set_aside_count` (for example "5 imported
  items left to the support and field producers, which own them") so it is visible, never silent.
- **Neither producer runs** (the usual case when this skill runs alone, or when the conductor's
  enabled members include neither): keep them, **labeled** as imported (for example "imported from
  your support tool"), so a standalone user keeps that signal. Label each quote taken from one the
  same way. If only one of the two producers runs, set aside only the items it owns (support-ingested
  or call-ingested, when the item shows which) and keep the rest, labeled.

**Telling call items apart.** Support-ingested items usually name their source (for example a
ticket tool). Call-ingested items often carry only an opaque source id (such as a UUID), so treat a
`-42` item whose source is not a recognizable support tool as call-ingested. Label it "call" plus
the account (for example "call, Acme"), in set-aside reasons, labels, and quote attributions alike,
and never present the id as a source name. The id still counts as the item's source reference for
the distinct-source counting in Step 4.

Always keep real visitor submissions and items one of your own employees entered by hand. Do this
**before** clustering: the list that remains after this step is the only input to Steps 3 to 5. The topic and insight tools cannot be filtered by author and
include system-ingested items, so never take counts, quotes, or themes from them directly.

**Only the `-42` system user is ever set aside by author, never a human author.** The one other
set-aside is noise (spam or an empty item), whoever wrote it, counted in `set_aside_count` with
its reason. Feedback authored by one of
your own employees is always counted: employees frequently log it **on behalf of a customer**
(attribute the item to that customer via `onBehalfOf` / `account`, not the employee), and employees
who use the product are power users whose own feedback is genuine signal. Count internal-authored
items rather than dropping them.

**Attribute each item to the customer it speaks for.** When an item names a customer in its "on
behalf of" field (`onBehalfOf` in Pendo Listen), attribute the item to that customer's account, and
note the account that submitted it (for example a partner, an agency, or your own team) as the
submitter, for example "Globex (submitted by Acme Partners)". Use that customer account, not
the submitting account, for `account_count`, the recurring-account check in Step 4, the ARR join in
Step 5c, provenance account keys, and quote attributions. Resolve the customer's account by exact
id: use it directly when the tool gives one; in Pendo Listen `onBehalfOf` is a visitor, so when it
differs from the item's own account, read that visitor's account from their other feedback items
(`getFeedbackItems` filtered by `visitorIds`) or from visitor metadata. If it can't be resolved by
id, keep the item's own account and say so; never infer an account from an email domain or a name.
When the field is empty or names someone in the submitting account itself, use the item's own
`account`.

**Machine-written items count, labeled.** Some kept items are written by an automated assistant or
bot (for example an in-app assistant summarizing a conversation) rather than typed by a person.
Identify them from the item's metadata where the tool exposes it (author type, source app, or an
author that is an assistant, bot, or service account), otherwise from clear text markers in the
item (for example "summary generated by the assistant"). Never guess: an item with no such signal
is treated as human-written. Machine-written items count toward themes, `item_count`, and
`account_count` like any other item, but each one is labeled "machine-written" wherever it appears:
in the theme's evidence line (for example "6 similar items (2 machine-written) · 4 accounts"), in
pattern bullets, and in the attribution of any quote taken from one. They also need a human-written
companion before a new idea is drafted (Step 5).
> Cross-producer de-duplication of the *same item* surfaced by two producers is a composer-layer
> responsibility: stamp every emitted theme with a `provenance` set of stable keys (see Step 6) and
> the composer merges on shared keys.

### 3. Cluster into themes and type them
Cluster the **kept** items from Step 2 yourself, reading each item's text. `generateFeedbackTopics`
(same filters) is an optional hint: it returns only `topicName`, `topicDescription`, and
`vocInsightCount` (no quotes, sentiment, or item ids), and its counts include system-ingested items.
Use its names and descriptions as candidate theme labels if they fit, then assign each kept item to
a theme yourself. Take every quote from the raw item text (`getFeedbackItems`), attributed per Step 2,
and derive each theme's sentiment from its items. Counts come from your own assignment, never from
`vocInsightCount`. (With pasted data, cluster the items the same way.) For each
theme, also assign **one type** (**Bug / Feature Request / Pain Point / Praise / Other**) from its
title and quotes. **Compute two evidence counts per theme, the number of feedback ITEMS and the
number of DISTINCT ACCOUNTS, and carry them through (rendered "N similar items · M accounts").**
Sort by volume, weighting recency (a theme that's mostly NEW this window ranks above an older steady
one). `getFeedbackInsights` can inform the one-line cross-cutting summary, but it also draws on
system-ingested items: ignore any insight that rests only on items set aside in Step 2, and never
quote from it.

### 4. Escalation scan (before prioritising by volume)
Flag any theme that signals urgency, for example an outage, a compliance/security concern, a
churn/enterprise blocker, or the same account recurring across distinct sources. These are examples,
not a closed list: judge urgency from the items themselves. One enterprise blocker outweighs
twenty feature requests; escalations lead the output regardless of volume.

**Same account recurring counts distinct sources, not items.** For each account in a theme, count
the distinct sources its items came from: separate calls, meetings, tickets, survey responses,
in-app submissions, or other conversations. Items that share a source id, source link, or source
record, or that were logged from the same call or meeting, count **once**, so one call logged as
several items is one source. When an item carries no source reference, treat items for the same
account by the same author on the same day as one source; otherwise count each such item as its own
source. The recurrence escalation needs the account to recur across two or more distinct sources.
This changes only the escalation check; `item_count` and `account_count` are counted as before.

### 5. Match top themes to ideas, and LINK
**Check existing links first.** Each item from `getFeedbackItems` carries `linkedIdeas`. Items
already linked to an idea are never linked again and never proposed as links; if most of a theme's
items point at the same idea, treat that idea as the theme's match (it is context, not a new ✅).
Only **unlinked** items are candidates for the linking below.

**Read the full idea backlog first.** Before matching any theme, read every idea in the theme's
product area. Read the ideas tool's schema first and use only the parameters it defines. If it has
paging or cursor parameters, call until no further ideas come back. If it caps results with no
paging (Pendo's `getIdeas` returns at most 30), read in filter slices: one call per status value
the tool supports (`getIdeas` lists them as `availableStatuses`, passed as `statusNames`), then
split any slice that hits the cap by date range (`startDate`/`endDate`), halving the range until
each slice is under the cap. Merge by idea `id`. The backlog counts as fully read only when the
merged count equals the area's total, when the tool reports one; otherwise, when every slice came
back under the cap. Check what the date filter actually filters on: if returned ideas were created
outside the requested range, it is not slicing by creation date and can skip ideas, so rely on the
count check. The cached `idea_index` counts only if it was built the same way and is current.
Then match each theme locally against that full list, reading each idea's title and description
against the theme's core ask. `getIdeas` similarity search does not reliably narrow results (it
can return the most recent ideas in the area, none on topic), so treat its output as a hint only,
never as proof that a match does or doesn't exist. An exact keyword-stem search can help surface
candidates.

**If the full backlog can't be read** (no paging and the slices don't add up, a call errors, or a
slice can't be split under the cap), still propose any existing-idea match found in the partial
list, but produce **no new-idea draft** for that run, for any theme. In place of the draft or the
"Create one?" question, say "couldn't check the full idea backlog (read N of M)", with N the
distinct ideas read and M the reported total (or "read N, total unknown" when the tool reports
none). Render it in the standalone digest and append it to `insight` in the structured output.
A match is **confident** only when all three hold: there is exactly one candidate idea, that idea
names the theme's core ask (the thing customers are asking for or reporting, not just the same
area), and no second candidate comes close. Anything else is not confident: treat it as multiple
plausible ideas or no match below. When a theme has a **confident** single match, link the 2 to 3 most representative **unlinked**
feedback items under that theme with `linkIdeaAndFeedback` (don't bulk-link every item). If every
item under the theme is already linked, there is nothing to link. Otherwise, how the links are applied depends on `auto_link`:

- If `feedback_idea.auto_link` is `true` (typical for scheduled runs feeding a daily digest), link
  without asking.
- Otherwise (the default, and the usual case when run on its own), list the proposed links and ask
  for one confirmation covering all of them before calling `linkIdeaAndFeedback`. Unconfirmed
  links go to 🟡 as `pick_idea` staged actions carrying the idea and feedback ids.

Log every applied link in the run output (idea id, feedback ids, theme) so it can be reverted with
`unlinkIdeaFeedback`. Report applied links as **✅ Handled for you**, each linked, for example
*"Linked 2 Timesheets items about locking approved weeks to the idea [Lock a timesheet once it's approved](idea_link)."*

Fall back to a **🟡** when judgment is needed: **no** candidate idea in the full backlog (*"no
matching idea for '{theme}'. Create one?"*; with a partial backlog, the "couldn't check" line
above instead), or **multiple** plausible ideas (*"'{theme}' could map to A or B.
Which?"*).

**Draft a new idea for a well-evidenced theme with no match.** Only when the full backlog was
read (above). When a theme has no matching idea and its items come from `feedback_idea.new_idea_min_sources` (default 3) or more **distinct sources**
(counted the same way as Step 4: items sharing a source id, link, or record, or logged from the same
call or meeting, count once), and at least one of those sources is human-written (Step 2), draft
the idea inside the 🟡 instead of the short question. A theme whose sources are all machine-written
gets the short question instead, with the reason. Draft contents:

- **Title** in the customers' own words (from the item text, not internal jargon).
- **Problem**: one or two sentences on what customers can't do or what hurts.
- **Who is affected**: the distinct account count, and the accounts' ARR where it resolves (the
  same exact-id account join as Step 5c; omit ARR when `arr_field` is unset or nothing resolves).
- **Representative quotes**: 2 to 3 real, attributed quotes, each linked to its feedback item.
- **Suggested product area**: the theme's area.

Emit it as a `create_idea` staged action carrying the `draft`. Nothing is created until the user
approves. On approval with Pendo MCP connected, call `createIdeaItem` with the draft (Pendo app and
the suggested area), then `linkIdeaAndFeedback` the representative items, and treat the new idea
exactly like a matched one for the ARR and hyperlink steps below. Without Pendo MCP, hand over the
draft as text for the user to create by hand in their feedback tool. A theme under the threshold
keeps the short "Create one?" question.

When the user approves creating an idea from the short question, call `createIdeaItem` (Pendo app
and the theme's product area), then `linkIdeaAndFeedback` the representative items, and treat the
new idea the same way.

**Autonomy boundary (do not blur this):** linking feedback to an *existing* idea may be automatic
(when `auto_link` is on) and is reported as ✅; **creating a new idea is never automatic**. It is
always a 🟡 the user approves first, and only after approval do you call `createIdeaItem`. Never
narrate a creation as something the producer did on its own.

### 5b. Connect the idea to its feature record: SUGGEST
Only if a feature tracker is configured. A matched Listen idea should map to a **feature record**
via the `feedback_idea.idea_field` property. For each idea that got a confident match in
Step 5, check whether a record in `feedback_idea.feature_tracker` already references it in that
field:

- **Already connected**: nothing to do; note it as context on the theme.
- **Not connected but a feature clearly corresponds** (same area plus title/topic match): stage a
  **🟡** suggestion: *"Idea '{idea}' isn't linked to a feature record. Connect it to '{Feature}'
  by setting {idea_field}? [ Connect ] [ Pick another ] [ Skip ]"*, carrying the feature
  record id, the idea id, and the target field.
- **No corresponding feature**: stage a lighter 🟡: *"Idea '{idea}' has no feature record yet.
  Create or link one?"*

The tracker write stays a **suggestion**, applied only on approval. The feedback-to-idea link is
Step 5; the idea-to-feature link is offered. If `idea_field` can't be resolved or no tracker
is connected, skip this hop and name the gap.

### 5c. Associated ARR (primary: native · fallback: account join)
For each matched or newly created idea, attach its **associated ARR** so themes can be weighed by
revenue, not just volume:

- **Primary (only if available):** if `getIdeas` returns an idea-level ARR field, read it directly.
  The current `getIdeas` response has no idea-level ARR, so expect the fallback below in practice.
- **Fallback:** gather the idea's associated accounts (the `accountId`s on its votes plus each
  linked feedback item's customer account, attributed per Step 2), then `listAccounts` (`select` = `feedback_idea.arr_field`),
  resolving each account **by its exact id**. Account names aren't unique; resolve by id and never
  fuzzy-match on name. Add up the resolved values, **leaving out accounts whose value is empty or zero**, and report
  `ARR resolved for N of M accounts`.
- **Concentration (dynamic):** compute the top account's share of the total; when it is at/above
  `feedback_idea.arr_concentration_threshold` (default 0.5), flag it (for example "most of the linked ARR
  sits with one account (about 60%), so the total isn't a broad signal") so the number isn't over-read.

If neither path resolves, or `arr_field` isn't configured, omit ARR rather than guess.

### 6. Emit the structured producer output
When a composer or orchestrator is present, or the user asks for it, hand over this compact object
(when run alone, render the standalone digest below and include this block only on request):

```
category: field_and_feedback           # -> 💬 From the field & feedback
headline_theme: { name, type, item_count, account_count, sentiment, delta, one_line (full sentence), quote, escalation: bool, source_link, provenance }
themes: [ up to ~5, each { name, type, item_count, account_count, sentiment, new_since_last_run, maps_to_idea: {id, title, idea_link, arr: {total, resolved_accounts, total_accounts, top_account:{name,share}, source:"native"|"computed"}|null}|null, feature_record: {id,title}|null, quote (real, attributed), patterns: [ { desc, quote, accounts, count } ], link, provenance } ]
insight: "<one-line cross-cutting summary>"
handled: [ { kind: linked_feedback_to_idea | created_idea, idea, idea_link, feedback_ids, theme, arr } ]                        # -> ✅ links done this run
staged_actions: [ { kind: connect_idea_to_feature | create_idea | create_feature | pick_idea, idea?, feature?, feedback_ids?, theme, deep_link, draft? } ]  # -> 🟡
                # draft (create_idea for a theme with new_idea_min_sources+ distinct sources): { title, problem, who_affected: { account_count, arr|null }, quotes: [ { text, attribution, link } ], suggested_area, source_count }
escalations: [ { theme, why, accounts, link } ]                                                             # -> 🔴 if truly blocking (a judgment; Step 4's list gives examples, not a closed list)
set_aside_count: <# items set aside because another producer covers them (Step 2), or as noise>
```

**Contract keys are stable and tool-neutral.** The field names above and the provenance key
prefixes below are the interface a composer reads, so keep them exactly as written whatever the
underlying tool: `feature_record` holds the matched feature record from whatever tracker you use,
`feedback_item:` / `idea:` hold the feedback and idea ids from whatever feedback tool supplied them
(for example Pendo Listen), and `account:analytics:` holds the account id from that tool.

**`provenance` (per theme/headline) is the composer's de-dup key**, the set of stable ids behind the
theme: `idea:<id>` (from `maps_to_idea.id`), each contributing `feedback_item:<id>` (the
Pendo MCP `getFeedbackItems` / `getIdeas` `id`s, up to the top few), and for each account **both**
`account:analytics:<id>` and, when `crm_account_id_field` is set and the account carries a value,
`account:crm:<crm account id>`. This is what lets a feedback theme correlate with other producers'
CRM-keyed accounts. The composer merges any item sharing a non-account key with another producer's
item (a ticket that became feedback, a call that became an idea), so always emit the real ids, never
fabricate them. With pasted data, a matched pasted idea that carries an id becomes an `idea:<id>`
key (and pasted feedback items with ids become `feedback_item:<id>` keys); an idea or item with no
id gets no key. With no ids at all, emit an empty provenance set rather than invented ids.

Keep it **discovery, not recap**: only what's new or escalating; exclude anything that came straight
to the user. If nothing is new, emit a single calm line that still carries **direction**: the
new-item count plus the delta vs the prior window (for example *"quiet in your areas: 3 new items this week,
about the same as last week; nothing to action"*). Never pad and never invent a theme; a quiet day is
a valid, useful answer.

**Rendering contract (for the composer and for the standalone digest):** write `one_line`, `quote`, and theme text as **complete sentences** (no truncation), and **always carry a real, attributed customer quote for the headline theme and every surfaced pattern. Quotes are the highest-value signal; never drop them in favor of bare account lists.** **Prefer genuine customer voice, but never drop, placeholder, or de-weight a quote for lack of it.** An employee-authored item counts toward evidence like any other. When it was submitted *on behalf of* a customer, attribute the quote to that customer, not the employee. When an employee is summarizing a customer's situation, an internal doc, or a contractual ask, and that summary is the best representation of the signal, **use the employee's own words**, attributed to them and what they're summarizing (for example *"Sam Okafor, account manager, relaying what Initech's operations team asked for on a renewal call"*). Only prefer a plain customer quote when one is genuinely stronger. **Do not disqualify or discount a quote because its underlying source is old**: staleness of a linked doc, ticket, or call is not a reason to drop or down-rank the signal. Never render a "no quote" placeholder and never fabricate one: always surface the single best available quote, customer or employee. **The `one_line` and the `quote` must complement, never restate each other**: the one_line carries the analytical framing (severity, scope, why it matters); the quote adds the primary-source voice and specifics the framing omits. If the best quote would just repeat the summary, compress the summary to a short setup and let the quote carry the substance. Never print two sentences that say the same thing. Each theme's **patterns render as bullets**, and pattern bullets are **customer-signal only** (what's happening + a real attributed quote + accounts + `N similar items`). The idea match, the link (✅), and the ARR render as a **separate status line beneath the theme headline, never as a pattern bullet**, so at a glance the bullets are unmistakably the customer signal. Every idea reference carries a **complete** `idea_link` (the full `feedback_idea.idea_url_pattern` with `{subscriptionId}` and `{ideaId}` both substituted, never a bare section URL like `…/listen`) so it renders as a **hyperlink**; if the pattern is unset or an id is missing, cite the idea by title, plus its id in plain text when known, rather than emitting a section-level stub. Every theme shows its **evidence count** ("N similar items · M accounts") and, where resolved, the idea's **ARR** with any concentration flag.

### 7. Standalone digest (when run on its own)
When no composer is present, render the output from Step 6 as a readable markdown digest in chat,
following the rendering contract above:

```
Feedback digest: <focus areas> · <date> · <window>
<one calm line on a quiet day, or a one-sentence headline naming the headline theme / escalation>

💬 From the field & feedback
**<Theme name>** (<type> · N similar items [(K machine-written)] · M accounts · <sentiment>)
 • <pattern: what's happening, "real quote" (attribution), accounts>
 Status: matches [<idea title>](<idea_link>) · linked 3 items · ~$ARR (N of M accounts resolved)
<one entry per surfaced theme; swept-but-quiet areas listed as "<area>: quiet">

Insight: <one-line cross-cutting summary>

✅ Handled for you: <links applied this run, each hyperlinked>
🟡 Needs your OK: <staged actions: create idea (with the draft for a well-evidenced theme), pick idea, connect idea to feature>
🔴 Needs you to act: <truly blocking escalations only>

Set aside N items (<reasons>).
Coverage: <items retrieved, date ranges, and whether any range hit the ~30-item cap>
Idea backlog: <read N of M per area; "couldn't check the full idea backlog (read N of M)" when partial>
```

Omit ✅ / 🟡 / 🔴 lines that are empty. One complete sentence per item. If
`feedback_idea.standalone_digest` is set, also post this digest to the configured channel on its
own cadence, in addition to showing it in chat. When a composer is present, this digest is not
rendered; the composer narrates the structured output instead.

## Scheduling (optional)

The skill runs on demand. To run it automatically, register it with whatever scheduler your Claude
environment provides (for example a daily or weekly scheduled task) with the prompt "Run
feedback-idea-producer for my focus areas". Daily is the default cadence; weekly (for example
Monday) also works. Scheduled runs are non-interactive: no mid-run prompts; links apply only when
`auto_link` is true, and every other decision is staged as a 🟡 for the next time the user opens
the chat. Update `last_run` at the end of every run.

## Degrade gracefully

- Pendo MCP not connected: use pasted or exported feedback (see Sources); if none, say so and stop.
  New-idea drafts are still written and handed over as text; nothing is created.
- `linkIdeaAndFeedback` fails: surface the ids for manual linking.
- `getFeedbackItems` hits its ~30-item cap even after splitting the window, or the coverage check
  finds a gap it can't close: report what was retrieved against the reported total and say the
  coverage is partial.
- The area's full idea backlog can't be read (filter slices don't add up to the total, an error, or a cap): propose
  existing-idea matches from the partial list, produce no new-idea draft, and say "couldn't check
  the full idea backlog (read N of M)".
- Feature tracker or `idea_field` can't be resolved: skip the feature hop and name the gap
  rather than guessing.
- Running under a composer and nothing is available: emit nothing and let the composer name the gap.

## Connectors (all optional)
- **Pendo MCP**: `listAllApplications`, `getFeedbackItems`, `generateFeedbackTopics`,
  `getFeedbackInsights`, `getIdeas`, `linkIdeaAndFeedback`, `unlinkIdeaFeedback` (revert),
  `createIdeaItem` (on approved create), `listAccounts` (ARR fallback).
- **Feature tracker** (Notion, Jira, Airtable, or similar): read records and the
  `idea_field`, and write the connection only on approval.
- **Slack** (or another channel): only if `standalone_digest` posting is enabled.

## Principles
- **Producer first**: emit structured signal; a composer narrates it when present, and the skill
  renders its own digest when it isn't.
- **Delta + discovery**: what's new since last run, escalations first, no recap, no already-seen.
- **Link feedback to ideas** (logged and revertible, automatic only when enabled); the
  idea-to-feature connection is *suggested*, never silent.
- **Weigh by revenue, not just volume**: attach each idea's associated ARR (native when available,
  else an exact-id account join) and flag single-account concentration.
- **Every idea reference is a hyperlink**: carry `idea_link` so the reader jumps straight to it.
- **Config-driven, not hardcoded**: focus areas, subscription, cadence, channel, feature tracker,
  the Listen-idea field, ARR field, and idea-URL pattern all come from the user's setup.
