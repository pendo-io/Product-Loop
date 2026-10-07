---
name: engineering-progress-producer
description: 'Produces a delta-only digest of engineering progress for a PM from any issue tracker: Jira, GitHub Issues, Linear, Azure DevOps, Shortcut, or a pasted export. Reads the PM''s boards, projects, or repos, pulls what changed since the last run, and surfaces what SHIPPED, what''s BLOCKED, what NEWLY BROKE, and what got DECIDED, customer-facing first, focusing on what the PM might have MISSED. Measures that movement against each feature''s description and goals, then renders a short digest (🔴 act; 🚢 shipped, in-flight, blocked, decisions; ⚠️ heads-up; 🟡 confirm) and, with a digest composer, a structured update. Single owner of the issue-tracker delta for the daily digest. Not for drafting release notes (use release-note-drafter) or composing the digest (use attention-radar). Runs on demand or daily; optionally drafts a weekly team-channel summary, posted only after approval. Triggers: "engineering progress", "what shipped", "what''s blocked", "what broke", "sprint progress", "jira digest", "github issues digest".'
license: MIT. LICENSE has complete terms
compatibility: Standalone. Works on a pasted tracker export or change list with no connectors. Optional, used if present - an issue tracker connector or CLI (Jira, GitHub Issues via gh, Linear, Azure DevOps, Shortcut), a feature tracker such as Notion, Jira, or Airtable (feature description and goals), Slack (weekly team summary, draft-first). Optional companions - attention-radar and the support-trends-producer hand-off.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

## When to use

Use for a PM's daily engineering progress from their issue tracker: what shipped, what's blocked,
what newly broke, what got decided. Triggers: "engineering progress", "what shipped", "what's
blocked", "what broke", "sprint progress", "jira digest", "github issues digest". Run it on demand or
daily, and optionally have it draft a separate weekly team-channel summary.

# Engineering Progress Producer

A **producer**: it owns one source (the team's issue tracker) and produces a short, **delta-only**
update each run.

- **Run alone**, it renders its own readable digest in chat (see "Standalone digest").
- **Run inside a daily digest**, it hands a structured update to a digest composer such as
  `attention-radar`, or an orchestrator that runs producers daily. The composer folds it into the
  digest's *existing* categories; this producer adds **no section of its own** there. Both skills are
  in https://github.com/pendo-io/Product-Loop. If neither is installed, you don't need them: the
  standalone digest is the complete output.

Its job: answer *"what shipped, what's stuck, what got decided, and what just broke for customers in
my areas that I might have missed, and is it moving my features toward their goals?"* without
recreating the tracker's UI.

## Done

- **Result:** a short engineering-progress digest in chat (Standalone digest), or, under a
  composer, the structured object in Step 6.
- **Done when:** every surfaced item carries a complete link (or the key in plain text in pasted
  mode), what changed, who (or "who: unknown"), and when; every board either produced items, the
  calm line, or a named gap; the structured block has the keys in Step 6.
- **Next consumer:** `attention-radar` or a daily-digest conductor, which folds items into its
  existing categories. Run alone, the person who asked.
- **Intent:** discovery, not a standup readout. When steps run out, surface the departures from
  plan and name what couldn't be checked.

## Boundaries and authority

Hard stops:

- Never invent issues; with no tracker and no paste, say so and stop.
- Never prompt on a scheduled run; emit nothing for a board missing a required piece and name the
  gap.
- Never guess who made a change; say "who: unknown" when the history doesn't show one.
- Never attribute a bot's change to a person or to eng, because a bot auto-closing or sweeping is
  not an eng decision; treat it as noise unless a human acted on the result.
- Never infer a product-ask from an ambiguous remark; treat it as a risk comment under the normal
  deviation test.
- Never read done-ness from a status name, because names mislead (for example a state called
  "Queued" can mean waiting to be started, not waiting to ship); use the tracker's state category.
- Never file an issue; stage the "file a bug or task" suggestion.
- Never post the weekly summary without the user's approval in chat; show the draft first.

**Authority.** Every write this skill can make:

- Its own `config.json` (`jira_progress` namespace): `boards_resolved` at setup or when a board URL
  changes, `epic_feature_map` on each confirmed mapping, `custom_fields` and
  `customer_facing_signal` at setup, `weekly_summary` when the user opts in, and other setup
  answers when the user agrees to save them.
- Slack or similar: the weekly team summary, only after the user approves the draft. Read it back
  by opening the returned message permalink before reporting it posted; if there is no permalink,
  say the post couldn't be confirmed.
- After any `config.json` write, re-read the file and confirm the saved value before reporting it.

Read-only on the issue tracker, the feature tracker, and `support_trends.eng_wait_queue`.

## Judgment vs mechanics

The tracker's search owns the window and scope (JQL, `gh search`, a Linear filter); its state
category (`statusCategory`, `state_reason`, state type) owns done-ness; the account type owns bot
detection where the tracker reports it. The model judges: deviation versus routine fulfillment,
product-ask versus remark, movement toward or away from a feature's goal, ranking, and phrasing.
These mechanics are still done by the model from prose and are candidates for code: the
working-day stall count, the candidate cap, compaction to one line per issue, and post-ship counts
per category.

## Honesty

- An empty result is valid: one calm line, never padded.
- Couldn't check is not nothing there: an unresolved board is a named gap; in pasted mode a miss
  means "no matching bug in the pasted data", not "no bug exists"; signals a paste can't support
  are named in one line.
- A missing north star is noted once, and movement is reported without that framing.
- No time estimates.

**Before presenting**, check: no issue key outside a link; no bot change attributed to a person; no
stale flag reported as a blocker; no product-ask inferred from an unaddressed remark; no done-ness
read from a status name. Fix what fails and check once more; if a check still fails, ship with the
failure named.

**Terms.** "PM" means whoever owns the product decisions for the work. "PL" (product lead) means
someone who owns a portfolio of areas with PMs under them. A **feature record** is wherever your
team keeps a feature's description and goals: a Notion database, Jira epics, Airtable, a doc, or
text you paste. An **issue** is a ticket, issue, story, or work item in any tracker. A **board** is
whatever scopes the PM's work: a Jira board, a GitHub repo or project, a Linear team or project, an
Azure DevOps area path or query, a Shortcut team or workflow. An **epic** is the grouping above an
issue: a Jira epic, a GitHub milestone, parent issue, or project, a Linear project, an Azure DevOps
feature or epic.

**Contract keys keep their Jira names.** The keys `jira:<KEY>`, `jira_key`, `jira_status`,
`linked_jira`, `queued_for_jira`, and the `jira_progress.*` config namespace are named for Jira for
compatibility with the composer and the support producer, and they apply to any tracker: they hold
that tracker's issue reference (for example `jira:<TEAM>-<n>` for Linear or `jira:owner/repo#123` for
GitHub Issues).

## Inputs: where the tracker data comes from

Use the first one available:

1. **A tracker connector or CLI** that can search issues updated in a window and read one issue's
   history (who changed what, when). Scope comes from board URLs (below). See "Reading your tracker"
   for per-tool notes.
2. **Pasted data.** A CSV/JSON export, a saved-query result, or a plain list of changes pasted into
   chat. Ask once: *"Paste your board's issues updated since {last run}. Include status, type,
   priority, assignee, labels, any blocked flag, and recent comments or history if you have them."*
   On the first run, when there is no `last_run` yet, ask instead: *"Paste your board's issues
   updated in the last 24 hours. Include status, type, priority, assignee, labels, any blocked flag,
   and recent comments or history if you have them."*
   Work from what is there, and say in one line which signals the paste couldn't support (for example,
   no history means no "who changed what" and no stall detection beyond the updated date).

If neither is available, say so and stop. Never invent issues.

## Config it reads (never hardcode identity)

All identity comes from config, seeded from **board URLs**. Store it in a `config.json` next to this
skill if the environment allows it (see `config.example.json`); otherwise ask for the pieces at the
start of the run and offer to save them.

- `jira_progress.tracker`: which tracker this is (`jira`, `github`, `linear`, `azure_devops`,
  `shortcut`, `other`, or `pasted`). Inferred from the board URL at setup and confirmed once.
- `jira_progress.board_urls`: one or more board URLs (a Jira board such as
  `https://<your-jira-site>/jira/software/c/projects/<PROJ>/boards/<id>`, a GitHub repo or project
  URL, a Linear team or project URL, and so on). From each, resolve the **site id** (the connector's
  site or workspace identifier, called cloudId on Jira, when the tracker has one), the **project**
  (a Jira project, a GitHub `owner/repo`, a Linear team), the **board id** if any, and the **scope
  query** in the tracker's own syntax (the Jira board's filter JQL, GitHub search qualifiers, a Linear
  filter). This is the primary scope input. **Resolved once and stored** in
  `jira_progress.boards_resolved` (`{cloudId, project, board_id, board_filter}` per board; `cloudId`
  is the site id (cloudId) on Jira and the workspace id or `null` elsewhere) so runtime never
  re-resolves. Refresh only if a board URL changes. When the daily-digest conductor passes
  `org_constants` (or its shared config exists at `<workspace>/.daily-digest/config.json`), read
  `org_constants.jira_site` first on Jira: it confirms the site each board URL points to and is the
  base for issue links (`<jira_site>/browse/<KEY>`). If a board URL names a different site, use the
  board URL's site for that board and say so once. Without it, take the site from the board URLs.
- `focus_areas`: the PM's product area(s), used to keep the read to their work and to find their
  feature records.
- `jira_progress.features_db`: where the feature records live (a tracker database URL, or `"pasted"`).
  Each feature's description and goals is the north star. Optional.
- `jira_progress.epic_feature_map`: the stored **epic → feature** associations. **Proposed, then
  confirmed, never silently assumed** (see Setup and Step 4): when the skill sees an epic on the
  board's issues that isn't yet in the map, it first tries to infer the right feature from the PM's
  focus-area features (matching the epic's name/summary against feature titles, descriptions, and
  goals) and asks them to **confirm or correct** its best guess: *"This epic looks like it maps to
  **{Feature}**, right?"* The confirmed answer is stored, asked once per epic, reused every run after.
- `support_trends.eng_wait_queue`: the optional cross-producer hand-off from `support-trends-producer`.
  **Definition (the same in every skill that touches it):** the support producer's `eng_waiting`
  entries with `queued_for_jira: true` and no `linked_jira` (support escalations waiting on engineering
  with **no linked bug**). The `support_trends` namespace is the support producer's own, whatever
  support tool it reads. If present, this producer reads it each run, looks each one up in the
  tracker, and either reports the now-existing bug or stages one to file (Step 5c). Read-only on the
  queue; the support producer owns writing it. If it's absent, skip Step 5c.
- `jira_progress.customer_facing_signal`: how a customer-facing issue is marked (a label, component,
  issue type, or field). Used for "customer-facing first."
- `jira_progress.custom_fields`: where this tracker carries the signals this skill reads, discovered
  once at setup (Setup step 4) because they differ per site: `flagged` (the blocked flag: a Jira
  custom field id, or a label or state name elsewhere), `customer_impact`, and, if the site has one,
  `support_ticket_ids` (a field holding linked support-ticket ids). Any signal the tracker doesn't
  have is `null` and is skipped with a one-line note.
- `jira_progress.stall_days`: how many **working days** with no history activity make an
  in-progress item "stalled" (Step 3). **Default 5**; configurable.
- `jira_progress.weekly_summary`: optional `{channel, day}` for the weekly team summary (Step 7),
  which is drafted and posted only after the user approves it; offered at setup. Default off.
- `last_run`: timestamp of the previous run, used for the delta window.

Every run takes its window and mappings from config (the conductor supplies `last_run` on
orchestrated runs), never from the conversation.

On a **scheduled run never prompt**: use config; if a required piece is missing, emit nothing for
that board and name the gap (the composer names it in the digest; standalone, say it in chat).

## North star: the feature's description and goals (not the sprint goal)

Framing is **not optional**, but it's **determined by the sources provided**: the yardstick is the
**feature record's description and goals** for the features the board's work maps to. For each
feature with activity this run, read its description/goals and judge whether the movement is *toward*
that goal, stalled against it, or a regression away from it. (A board URL can't return a sprint goal,
and the feature record is the truer statement of intent.) If a feature has no description/goals, or
no feature records are available at all, fall back to plain "shipped/blocked/broke" reporting and note
the missing north star once.

## Setup (once, then refresh)

1. **Capture board URL(s)** (or confirm pasted mode) and confirm the tracker. With a connector or
   CLI, resolve `{cloudId, project, board_id, board_filter}` per board and **store it in
   `jira_progress.boards_resolved`**, so runtime reuses it and never re-resolves (refresh only when a
   board URL changes).
2. **Point at feature records (optional).** Ask where feature descriptions and goals live: a connected
   tracker (store its URL in `features_db`), pasted text, or none.
3. **Seed the epic → feature map.** List the epics currently on the board's issues and, for each one
   not yet mapped, **infer the most likely feature** from the PM's focus-area features (epic
   name/summary vs. feature title/description/goals) and present it for **confirmation**: *"This epic
   looks like **{Feature}**. Confirm, pick another, or none?"* Store each confirmed answer in
   `jira_progress.epic_feature_map`. **Never assume a feature-key field or silently accept the
   guess**: the guess is a first offer the user confirms; new epics that appear later get the same
   propose-then-confirm on the run that first sees them (Step 4).
4. **Confirm the customer-facing signal** (label/component/type/field) once, and **discover where the
   other signals live** (the blocked flag, customer impact, a support-ticket-ids field). These vary by
   site, so never assume one; see "Reading your tracker" for how on each tool. Confirm them with the
   user and store them in `jira_progress.custom_fields`.
5. **Offer the optional weekly team summary.** Ask whether they want the weekly team-channel rollup
   (Step 7); if yes, store `{channel, day}` in `jira_progress.weekly_summary`. Default off.
6. **Scheduling is optional.** If the environment supports scheduled tasks, offer a daily run;
   otherwise the user runs it on demand.

## Daily workflow

### 1. Resolve scope
For each board URL, use the cached `{cloudId, board_filter}`. The board's scope query is the scope.
In pasted mode, the paste is the scope; filter it to `focus_areas` if it spans more.

### 2. Delta pull: two stages, cheap first
**Never pull per-issue history for every issue; that is what makes a run slow and large.** Do it
in two stages:

**Stage 1: one cheap search.** Search the board's scope **AND updated since `{last_run}`** (absolute
date from `last_run`; first run ≈ last 24h). Ask for summary, status (and its category), type,
priority, assignee, reporter, created, updated, resolution or close reason, labels, and comments,
**plus the stored signal fields from `jira_progress.custom_fields` in the same call.** **Keep each
page small.** Tracker search tools often enforce a minimum page size, so a small page is not always
available:
- **Request only the fields you need**, never all fields. If a page still comes back too large, drop
  comments from the search and read them only for Stage 2 candidates.
- **Use the tool's minimum page size.**
- **Narrow the query so a page stays small**: the updated window, the board's project, and the issue
  types that matter; on a wide window, split it into shorter updated ranges and run each as its own
  search.
- **Summarize each page before fetching the next**: compact it to one line per issue (below), keep
  only the compact lines, then request the next page.

The search still covers the whole delta; **no per-issue fetch yet.** NEW = created ≥ last_run;
UPDATED = older but touched in-window.

**Stage 2: history only for CANDIDATES.** From Stage 1, cheaply pick the handful that even *might* be
a deviation: status moved, blocked flag on/off, has a **new comment by someone other than the PM**,
customer impact set, a blocked-by link, closed as not done (Won't Do, not planned, canceled), or looks
stalled, and is **not** a routine closed-on-plan item. Fetch the issue with its history **only for
those candidates** (to get *who* changed *what*), with a **soft cap of about 10 to 15**; skip it
entirely for the obviously routine majority. On a normal daily (24h) window this is a handful of
issues; the wide back-fill window is the only case that is ever large.

Tracker output is often extremely verbose (even ~10 issues can exceed the context limit). If your
environment supports subagents, **hand the raw output to a subagent** that returns one compact line
per issue (`key · type · status · NEW|UPDATED · summary`, a `changes[]` of {who, when, what}, and the
latest 1–2 comment snippets with author), so raw output never reaches the digest. Give the subagent
the two-stage instruction above so it does **not** brute-force history across the whole set. Without
subagents, do the same compaction yourself before Step 3.

In pasted mode, skip the pull and compact the paste the same way.

### 3. Discovery filter: only what they'd miss or that builds a trend
Keep only issues the PM **might have missed** or that **contribute to a trend forming across other
producers** (a customer-facing break that matches a 💬 feedback theme; a blocked GA behind a 📈
adoption stall), when those producers ran. **Exclude** what came straight to the PM: issues they're
the assignee/driver on and already acting on, routine auto-transitions (for example auto-resolved on
merge with no story). Discovery, not a standup readout.

**Discovery is a DEVIATION from the plan, not the normal fulfillment of it.** The PM's job is to write
ticket *shells*; engineering's job is to expand, change, and **complete** them. The PM will never
close those tickets, eng will. So "eng refined / advanced / finished a ticket the PM created" is the
**expected division of labor, not a discovery**; surfacing routine on-track progression of the PM's
own planned work would flag every normal ticket as news. Establish the PM's **engagement watermark**
(their most recent history entry or comment) and look at what others changed after it, but that
diff only produces **candidates**. The decisive test is: *is this a departure from what the PM would
expect, or work the PM never planned?* Surface those; set the routine fulfillment aside silently.
What counts as a deviation / unplanned, ranked **bad-news and customer-facing first**:
- **Status regression**: a done or closed item reopened, or moved back toward to-do.
- **A blocker set or lifted by eng**: see the blocked-flag note below; also a new *"blocked by"* link,
  or a comment naming a dependency/slip.
- **Customer impact flipped on**, or a new customer-facing bug filed by eng/QA/support in the PM's area.
- **Re-prioritization**: pulled into (or dropped from) the active sprint or cycle, rank raised,
  priority raised.
- **Risk language in a new comment** the PM hasn't reacted to (*"won't make GA," "harder than expected,"
  "not safe to ship yet," "regression," "P0"*).
- **A brand-new issue the PM didn't create** in their area, **but** a routine planning issue/epic
  spun up by the PM's own program manager or eng is expected workflow; prune it. Surface a net-new item
  only when it's unexpected, customer-facing, or cross-team.
- **A product-consequential DECISION landed (high value: notify, don't accuse).** A change with
  **customer, external, or compliance** implications was made: a scope **cut** or an epic
  **descoped**, a customer-facing bug or story **closed Won't Do** (or not planned, or canceled), or
  a committed item **moved to a later release**. It isn't an explicit *ask* (so not the always-surface 🔴); surface it
  under **🚢 Shipped & in-flight** as a factual heads-up worth the PM's eyes, standalone and composed
  alike (emitted to a composer under `decisions`, Step 6). State **what changed, which issue, who made
  the change, and when**, factually, for example *"[The bulk-edit story](https://<your-tracker>/<issue>)
  was descoped from the Recurring tasks epic by Jordan on Wednesday."* (the issue key appears only
  inside the link, never in the sentence). When there is no history to name who made it (pasted mode,
  or a tracker that doesn't expose it), still report the decision and say **who: unknown** (*"was
  descoped on Wednesday; who made the change isn't in the data"*); never guess a name. **Do not editorialize** ("without
  you," "behind your back," "should have asked"): report the decision and let the PM judge whether
  they want to weigh in. This is often the richest signal on the board: not *is it blocked*, but
  *what got decided.*
- **Eng is explicitly asking Product for review or input: always surface this (the one hard rule).**
  Requires a *direct, product-directed ask*: a **needs-product-input / awaiting-product flag or
  label**, an **@-mention of the PM**, or a comment that clearly **requests product to review, decide,
  or sign off** (*"@PM can you confirm…", "need product's call on X", "waiting on product before we
  ship"*). This overrides the routine-fulfillment set-aside → **🔴 Needs you to act**, whether or not
  the work is on-plan, **unless the issue is closed/resolved** (the ask is moot). Every other signal
  in this list is still subject to the deviation test; this one is not.
  - **Do NOT infer an ask from ambiguous language.** A generic remark that something *should* be
    reviewed or reworked (*"the CSV totals look off for a couple of edge cases, worth someone
    sanity-checking"*) is **not** a product-ask: no named addressee, no request directed at product. Never
    fabricate the ask or claim eng is "waiting on you." Treat such a remark as a **risk comment** under
    the normal deviation test: surface it only if the item is itself a launch-critical / customer-facing
    deviation, and at a softer register (*"eng noted the CSV totals may be off for some edge cases"*), never as
    the always-surface product-ask.
- **Stall / silence near a deadline.** An in-progress item with **no history activity for
  `jira_progress.stall_days` working days (default 5)** while its target date / sprint or cycle end /
  GA approaches: nothing changing *is* the news when it should be moving. Default register is a soft
  *"you should take a look at this,"* **not** a siren; escalate to **🔴** only when the stalled item
  is **critical to a launch and has gone quiet near that launch**.
- **A cross-team dependency**: the PM's feature **blocked by an issue owned by another squad**. Emit
  it as a **candidate marked composer-suppressible**: the PM likely already knows about their own
  blocked-by link, so a composer drops it if the PM's meeting notes, Slack, or weekly update (which a
  composer may read via its own call-notes source setting) show they've already engaged with it.
  Surface only the dependencies the PM has *not* been talking about elsewhere. Standalone, with no
  such notes available, show it and mark it "you may already know about this."

**Set aside the routine; surface the departures.** The PM's own planned work advancing on-track
(shell → refined → built → done, the way it was meant to) is set aside silently; that's eng doing its
job, not news. **One nuance:** a *milestone* the work reaches (a **GA, a launch**) may be included as
brief **context that frames a real deviation** (for example *"Recurring tasks reached general
availability last week, and its follow-up story on skipped dates was just moved out of the sprint"*),
because the milestone is what makes the
deviation legible. Include it as grounding, never as standalone "you shipped this" news. **Bugs are the
clearest kind of unplanned work**: a bug and its fix are usually **eng-filed and eng-handled end to
end**, work the PM never planned or wrote a shell for, so a fix that shipped that the PM never worked
(**especially a customer-facing one**) is exactly the kind of thing they'd miss. The line to hold:
*did this depart from the plan or arrive unplanned?* → surface; *is this just the expected completion
of what the PM already set in motion?* → set aside.

**Weight by deadline proximity and cross-producer corroboration.** Rank against the feature record's
target date, the sprint or cycle end, or a known GA date: a blocker or stall on something **due this
week** outranks one due next quarter (a blocker two days before a launch matters more than the same
blocker a month out). And a
customer-facing break weighs **more** when it corroborates another producer: a tracker bug that
matches a live 💬 call/feedback theme or a 📈 adoption dip is more urgent than one raised in
isolation; carry the cross-link both ways.

> **Workflow realism: don't rely on a status named "Blocked."** Many workflows have **no Blocked
> state**; the blocker signal is a **blocked flag** (Jira's Flagged field, a `blocked` label, a
> blocked relation) plus a comment saying so (for example *"flagging this until the API change it
> depends on is merged"*). Read the blocked flag and blocked-by links, not just
> the status name, or you will miss the most important item on the board. **A flagged item that is
> resolved/done is a *stale flag*, not a live blocker**: the work is closed; note it as housekeeping
> ("flag left on a closed issue") or skip it, never surface it as blocking.
>
> **Read status MEANINGS, not the label's English.** Workflow states are often named in ways that
> mislead a literal read: in some workflows **"Queued" is a to-do state (lined up to be started,
> not lined up to ship)**, and a state like "Done (dev)" can still be waiting on QA. Use the
> tracker's **state category** (Jira `statusCategory`,
> Linear state type, GitHub open/closed plus close reason; see "Reading your tracker") as the ground
> truth for whether something is actually done or still pending; never infer done-ness or
> who-did-what from the status *name*.

> **Author realism: separate humans from bots.** History authors include **automation** (workflow
> automation, scheduled backlog sweeps, bulk clean-ups, CI and sync integrations).
> **Detect bots two ways**: the account type the tracker reports (for example Jira `accountType: app`,
> GitHub `type: Bot` or a `[bot]` login suffix), **or** the display name matches a bot pattern
> (contains "bot", "automation", or "integration", or names a connected tool such as a support-desk
> integration). Account type alone misses some: an integration account can look like a normal user.
> When unsure, don't attribute the change to a person. A bot auto-closing, auto-resolving on merge, or
> sweeping the backlog is **not** an eng *decision* and must never be attributed as one ("eng closed
> X" when a bot did). Treat bot/automation transitions as **noise**: surface them only when a
> **human** then acted on the result (for example a person reopened the bot-closed issue) or the
> bot's change left a real state a human must address. Attribute every surfaced decision/ask to the
> **human** who made it, by name, or say who: unknown when the data doesn't show one.

### 4. Map to features and measure against the north star
Map each surviving issue to its feature **via its epic**, using the stored
`jira_progress.epic_feature_map`. **If an issue's epic isn't in the map** (a new epic on the board),
infer the most likely feature from the PM's focus-area features (epic name/summary vs. feature
title/description/goals) and stage a **🟡** that **proposes** it: *"This epic (**{epic}**) looks like
it maps to **{Feature}**. Confirm, pick another, or none?"*, carrying the epic key, the suggested
feature, and the other candidates. Store the confirmed answer so it's never asked again, and report
that epic's issues without north-star framing until it's confirmed (don't act on the guess
unconfirmed). If there are no feature records at all, skip the mapping question and report without
north-star framing.
For each mapped feature, read its description/goals and characterise the movement: **toward goal /
slipping / blocked / regression**, using comment snippets to say *why* it moved.

### 5. Sort into the digest's EXISTING categories (no new section)
Tag each item to a category the daily digest already has. Do **not** invent a tracker block:

- **🚢 Shipped & in-flight**: shipped/merged/GA'd, moving, and blocked items, each with the
  feature's goal-context ("moves {Feature} toward its goal" / "blocks it"), plus
  **product-consequential decisions** (Step 3: a scope cut, a descoped epic, a bug or story closed
  Won't Do (or not planned, or canceled), or a committed item moved to a later release), emitted
  under `decisions` and stated as a factual heads-up.
- **🔴 Needs you to act**: a **customer-facing break that is live and hurting customers now** (a fix
  that merged but customers still see it; a new customer-facing bug customers are hitting), **and
  anything where eng is waiting on the PM** (needs-product-input / an unanswered @mention: eng needs a
  product answer to proceed).
- **⚠️ Heads-up** (conditional): a **customer-facing regression to watch** (not yet hurting
  customers, or not confirmed live), plus cross-team dependencies.
- **⚠️ Conflicts** (conditional, a separate category from Heads-up): the work collides with another
  effort or crosses a boundary.
- **Post-ship quality readout (for a just-shipped / just-GA'd feature).** Don't list every incoming
  bug: **categorise** the new eng-filed bugs against that feature (for example by component/symptom:
  timezone handling / approval routing / CSV export) and report **counts per category** ("since launch:
  7 new bugs, 3 timezone handling, 2 approval routing, 2 CSV export"), so the PM sees at a glance what
  worked and what didn't after launch.
  This readout goes under ⚠️ Heads-up.
- **Cross-link** a shipped fix or new break to the matching 💬 field/feedback theme, 🧭 competitor
  release, or 📈 adoption signal, reciprocally, and let corroboration raise its weight (Step 3).

**For a Product Lead, run in EVIDENCE MODE: no section of your own.** A PL doesn't work at ticket
level; a flagged issue on its own is noise to a leader. So for a PL, this producer does **not** get its
own bullets in a composed digest. It emits its full signal as **corroborating evidence** for the
composer, carrying strong **cross-link hooks** on every item (account/customer, feature, theme
keywords, and any commitment or launch name) so the composer can match a tracker fact to a finding
from another lens (a 💬 feedback theme, a 📈 adoption stall, a field/call signal, a tracked
commitment) and let that evidence *strengthen* the leader-level story. Only one thing may still
surface **solo** for a PL: a **commitment or launch at risk with no other corroboration**, the thing a
leader must know even if nothing else flags it. Everything else is handed over as support, not
surfaced on its own; the **composer owns whether and how a PL sees it**. Standalone for a PL, see
"Standalone digest."

### 5c. Reconcile the support eng-wait queue (optional cross-producer hand-off)
Skip this step if `support_trends.eng_wait_queue` is absent or empty.
Read `support_trends.eng_wait_queue`: the support producer's `eng_waiting` entries with
`queued_for_jira: true` and no `linked_jira` (escalations waiting on engineering with no linked bug).
For each, search the tracker for a matching bug: first by the ticket id in the site's
support-ticket-ids field when `jira_progress.custom_fields.support_ticket_ids` is set (an exact match),
then by the ticket's account/feature/summary: if one now exists, **report its status** (moving? stuck?
how long?) so the composer can merge it with the support escalation into **one 🔴**; if none exists,
the escalation is **shown once, as a 🔴**, carrying a **staged suggestion to
file the right kind of issue** as its action (the suggestion is not a separate 🟡 item): a **bug**
only if the defect is live/observable, or an **eng verification / hardening task** if it resolved but
is unverified (no bug without an observable defect), linked to the feature. Carry the support
wait-time / SLA breach and the ticket's `link` (its own URL, or null) through from the queue entry,
so the composer can deep-link the reconciled item. This is the other half of the support hand-off:
the digest shows the escalation once, not twice. This producer never files the issue itself; the
file suggestion is staged, never executed. In pasted mode, a miss means "no matching bug in the
pasted data," not "no bug exists"; say which.

### 6. Emit structured output for a composer (delta-only; no headline, no own section)
When a composer or orchestrator is present, or the user asks for it, hand over a compact object keyed
to the existing categories; the composer decides placement and density.
Each surfaced item carries **what changed, who changed it, and since when** (the change the PM missed
relative to their engagement watermark) so the composer can say *why* it's here, for example *"eng
reopened this after QA found the totals wrong again, two days after you signed it off."*

```
shipped_in_flight: [ { key, summary, status, feature, goal_context: "toward|slipping|blocked", link, new_since_last_run } ]   # -> 🚢
customer_facing_breaks: [ { key, summary, feature, why_it_matters, link } ]                                                    # -> 🔴 if live and hurting customers now / ⚠️ if a regression to watch (why_it_matters says which)
needs_you: [ { key, summary, kind: "awaiting_product_input|unanswered_mention", who_waiting, link } ]                          # -> 🔴 (eng is blocked on the PM)
eng_wait_reconciled: [ { support_ticket, link, account, jira_key: "<KEY> | null", jira_status, days_waiting, sla_status, action: "report|file_bug" } ]  # -> merges with the support escalation into one 🔴; link = the support ticket's own URL from the queue entry, or null
stalled: [ { key, summary, feature, days_idle, due, link } ]                                                                   # -> 🚢 (slipping) / 🔴 if deadline-critical
post_ship_regressions: [ { feature, since_ship, total, by_category: { "<cat>": <count> } } ]                                   # -> ⚠️ Heads-up (post-ship quality readout)
cross_team_dependencies: [ { key, blocked_by, other_team, composer_suppressible: true, link } ]                               # -> ⚠️ Heads-up unless PM already engaged elsewhere
conflicts: [ { key, with, boundary, link } ]                                                                                   # -> ⚠️ Conflicts (conditional)
decisions: [ { key, summary, resolution, feature, goal_context: "toward|slipping|blocked", who, when, link, new_since_last_run } ]  # -> 🚢 (factual heads-up; leader-level at PL altitude)
cross_links: [ { jira_key, related: "feedback_theme|adoption_signal|competitor_release", note } ]   # competitor_release: the item bears on a competitor move the competitor-release producer reported
staged_actions: [ { kind: map_epic_to_feature, epic_key, suggested_feature, candidate_features } ]                              # -> 🟡 (new epic: propose best-guess feature to confirm)
provenance_note: each item's `key` IS its `jira:<KEY>` provenance for the composer (on any tracker, the tracker's issue ref); `eng_wait_reconciled` also carries `support_ticket` + `jira_key`, and `cross_links` carry `jira_key`. The composer dedups on these (a tracker issue linked from a support ticket collapses to one item). See the attention-radar provenance-key rule.
set_aside_count: <# the PM's own driven progress + routine + already-seen-by-PM issues, not surfaced>
```

**`decisions`** carries the product-consequential decisions from Step 3: a scope cut, a descoped
epic, a bug or story closed Won't Do (or not planned, or canceled), or a committed item moved to a
later release. `resolution` names the decision in plain words (for example `"scope cut"`,
`"descoped"`, `"Won't Do"`, `"not planned"`, `"moved to a later release"`); `who` is the
human who made it (never a bot, per author realism), or `"unknown"` when there is no history to
show who (the decision is still reported); `when` is the date. State it factually, never
editorialized. It goes to 🚢 Shipped & in-flight everywhere: composers map it there, and the
standalone digest renders it there too.

`support_ticket` is the support-ticket id from the queue entry, whatever support tool produced it.
`link` is always a complete deep link to the issue (for example
`https://<your-jira-site>/browse/<KEY>` or `https://github.com/<owner>/<repo>/issues/<n>`); in pasted
mode with no site known, give the key in plain text instead of a partial URL.

Running daily keeps each increment small, so this shouldn't add noise. If nothing qualifies, emit one
calm line ("nothing new shipped or blocked in your areas"); never pad.

### Standalone digest (when run alone)
When no composer is present, render the same items as a short markdown digest in chat. Headers only
for categories that have items; one complete sentence per item, with the link on the phrase that
names the issue; customer-facing and bad news first within each group.

```
**Engineering progress: {focus areas} · {date} · since {last run}**

🔴 **Needs you to act**
- Eng is asking for your call on [how skipped dates should show in a recurring series](link): Jordan @-mentioned you yesterday and the story is waiting on product.

🚢 **Shipped & in-flight**
- [The holiday-skip rule for Recurring tasks](link) was reopened by QA on Monday after passing review last week, which sets Recurring tasks back against its goal of fewer manual reschedules.
- [The bulk-edit story](link) was descoped from the Recurring tasks epic by Jordan on Wednesday.

⚠️ **Heads-up**
- [Recurring events in Calendar Sync](link) is blocked by an issue another squad owns; you may already know about this.

🟡 **Confirm**
- This epic ([Series editing](link)) looks like it maps to **Recurring tasks**. Confirm, pick another, or none?

Set aside {set_aside_count} routine or already-seen issues.
```

Product-consequential decisions go under 🚢 Shipped & in-flight, as in a composed digest. A
customer-facing break goes under 🔴 when it is live and hurting customers now, and under ⚠️ Heads-up
when it is a regression to watch. "⚠️ Heads-up" also holds post-ship regressions and cross-team
dependencies; conflicts go under their own "⚠️ Conflicts" heading. If a support hand-off was reconciled, show each escalation once under
🔴: with its bug status when a matching bug exists, or as "no linked bug" with the staged suggestion
to file one when none does. If nothing qualifies, print the one calm line. Offer the structured block
after the digest ("want the structured output too?") unless the user asked for it up front.

**For a PL running it alone:** lead with the commitments or launches at risk, in plain
commitment/risk language without ticket jargon, capped at **1 per product area** when the PL covers
several areas and **2 to 3** when there is a single area. Include only items worth a leader's
attention; zero is fine, and then say in one line that nothing is at risk. Follow with a short
"Supporting evidence" list of the remaining items for drill-down.

### 7. (Optional) WEEKLY team summary, separate from the daily digest
Only if `jira_progress.weekly_summary` is set: **once a week** (its configured day), **draft** a
richer narrative rollup for the team channel: sprint/period framing, per-initiative narrative with
issue links as receipts, and a "Net" line. **Show the draft in chat and post it only after the user
approves** (and only if a Slack or similar connector is present; otherwise the draft is the output).
This is the team-facing artifact and is **weekly**, distinct from the daily feed (which stays
lightweight). It reads the same data; it does not run daily.

## Reading your tracker

The steps above are the same on every tracker. These notes say where each signal lives. Check exact
tool and field names against your connector; anything a tracker doesn't expose is skipped with a
one-line note, never guessed.

**Jira** (any Atlassian/Jira connector).
- Resolve the site id (cloudId) from the site (for example `getAccessibleAtlassianResources`). The
  board URL gives the project and board id; the board's filter JQL is the scope.
- Stage 1 is a JQL search: board filter AND `updated >= "<last_run>"`, narrowed by project and
  `issuetype in (Bug, Story, Task, Epic)`, requesting only `summary, status, issuetype, priority,
  assignee, reporter, created, updated, resolution, labels, comment` plus the stored custom field
  ids. Some connectors accept only a `maxResults` of 50 or more, so 50 is the smallest page. On a
  busy project a full page can exceed a typical tool-result limit even with few fields; trimming
  fields alone may not fix it, and splitting the `updated` window does.
- Stage 2 fetches one issue with its changelog (for example `getJiraIssue` with `expand=changelog`).
- Custom field ids (Flagged, Customer Impact, a support-ticket-ids field) vary by site, so never
  assume one. At setup, fetch one issue with field
  names expanded (`expand=names`, or `fields: ["*all"]` on that one issue), find the ids whose names
  match, and store them in `jira_progress.custom_fields`.
- Use `statusCategory` (`To Do` / `In Progress` / `Done`) for done-ness. Decisions show as a
  resolution such as Won't Do or Not a Bug.
- Bots: `accountType: app`, and the name pattern, since an integration account can be
  `accountType: atlassian`.
- When listing projects for a read, ask for projects you can **browse** (some connectors default to
  ones you can *create* in, for example `getVisibleJiraProjects` with `action: create`).
- Issue links: `<jira_site>/browse/<KEY>`.

**GitHub Issues** (the `gh` CLI or a GitHub connector).
- Scope is a repo (`owner/repo`), optionally narrowed by labels, milestone, or a project. Stage 1 is
  one search with qualifiers, for example `gh search issues --repo <owner>/<repo> --updated
  ">=<date>" --json number,title,state,labels,author,assignees,createdAt,updatedAt,url` or `gh issue
  list --repo <owner>/<repo> --state all --search "updated:>=<date>"`. `gh search issues` leaves out
  pull requests unless you pass `--include-prs`; in a raw search query add `is:issue`.
- State category is open/closed plus the close reason: the issue's `state_reason` (`gh issue view
  <n> --json stateReason`) is `completed` (shipped or done), `not_planned` (a decision, `resolution:
  "not planned"`), or `reopened`. Read it from the issue, because the timeline's `closed` event can
  carry `state_reason: null`. Search finds decisions with `is:closed reason:"not planned"`; exclude
  spam or triage labels the repo uses (for example `-label:<spam label>`), since closing spam is not
  a product decision.
- Stage 2 reads the issue timeline (`gh api repos/<owner>/<repo>/issues/<n>/timeline`): `closed`
  (with `state_reason`), `reopened`, `labeled`/`unlabeled`, `assigned`, `milestoned`,
  `cross-referenced` (a linked pull request), and `commented`, each with its `actor` and time. A
  timeline is verbose (even a few dozen events can exceed a tool-result limit), so compact it to
  {who, when, what} before keeping it.
- Labels carry most signals (blocked, customer-facing, needs-product); milestones and projects act as
  the epic and the sprint or cycle. Bots have `type: Bot` or a `[bot]` login suffix.
- Issue links: `https://github.com/<owner>/<repo>/issues/<n>`; the provenance key is
  `jira:<owner>/<repo>#<n>`.

**Linear** (a Linear connector).
- Scope is a team or project; Stage 1 lists issues updated since `last_run`. Each state has a type
  (`triage`, `backlog`, `unstarted`, `started`, `completed`, `canceled`) to use as the state category;
  `canceled` is a decision. Cycles are the sprint, projects the epic. Read issue history only if the
  connector exposes it; otherwise who changed what is unavailable and decisions carry who: unknown.

**Azure DevOps.** Scope is an area path or saved query; Stage 1 is a WIQL query on
`[System.ChangedDate]`, Stage 2 reads the work item's updates, and the state category comes from the
work item type's state categories (Removed is a decision).

**Shortcut.** Scope is a team, workflow, or epic; search stories updated since `last_run`, use the
workflow state type (unstarted, started, done) as the state category, and read story history where
the API or connector exposes it.

**Any other tracker, or a pasted export.** Anything that exposes issues, statuses, and some history
works: map its states to to-do / in progress / done (plus closed-not-done for decisions), use its
labels or fields for the blocked flag and customer-facing signal, and say in one line which signals
are unavailable (for example, no history means no who-changed-what, no engagement watermark, and
stall detection from the updated date only).

## Orchestrator notes (only when run by an orchestrator)
- **Class:** per-user scheduled **producer**. **Cadence:** **daily**. The optional team summary is a
  separate **weekly** draft, not a second daily run.
- **Digest buckets:** 🚢 Shipped & in-flight (primary), 🔴 for a customer-facing break that is live
  and hurting customers now and for eng waiting on the PM (`needs_you`), ⚠️ Heads-up (conditional)
  for a customer-facing regression to watch, for post-ship regressions (`post_ship_regressions`) and
  for cross-team dependencies, and ⚠️ Conflicts (conditional, a separate category) for `conflicts`. The only 🟡 it raises is the *"what feature is this epic?"*
  mapping question for a **new epic** (the answer is stored in `epic_feature_map`). A support
  hand-off with no matching bug is one 🔴 carrying a staged "file a bug/task" action.
  Product-consequential decisions go to 🚢 via `decisions`. Writes: see Boundaries and authority;
  nothing is written to the issue tracker or the feature tracker.
- **Non-interactive on scheduled runs.**
- **Degrade gracefully**: if no tracker connector is present, or a board URL / site id won't resolve,
  emit nothing for it and name the gap; if feature records are missing, report movement without the
  north-star framing and note it.

## Connectors (all optional)
- **Issue tracker** (Jira, GitHub Issues, Linear, Azure DevOps, Shortcut, or any connector or CLI
  that can search issues and read their history): see "Reading your tracker." Read-only. Without one,
  use pasted data.
- **Feature tracker** (Notion, Jira, Airtable, or similar): read feature descriptions, goals, and
  target dates. Read-only. Without it, use pasted text or skip the north-star framing.
- **Slack** (or similar): only for posting the optional weekly team summary, after approval.

## Principles
- **Producer, not composer**: emit signal tagged to the digest's *existing* categories; add no section
  to a composed digest. Alone, render a short digest of your own.
- **Discovery, not a standup**: only what the PM missed or what builds a cross-skill trend;
  customer-facing first.
- **Fast by default**: one cheap search for the whole delta; pull per-issue history only for the few
  candidates (soft cap of about 10 to 15); the daily 24h window keeps every run small. Never
  brute-force history across all issues.
- **Report, don't accuse**: state what changed and who made the change, factually and neutrally; this
  producer *notifies*, it does not indict. Never frame a change as eng going around the PM ("without
  you," "behind your back," "should have asked"). Surface the fact; the PM draws their own conclusion.
- **North star = the feature's description and goals**: measure movement against intent, sourced from
  what's connected or pasted.
- **Config-driven, not hardcoded**: tracker, boards (by URL, resolved once and stored), feature
  records, customer-facing signal, and the weekly channel all come from the user's setup.
- **Propose, then confirm the epic→feature mapping**: infer a best-guess feature from the PM's
  features and have them confirm the first time an epic appears (never silently assumed); store it and
  reuse. Resolved board metadata and the map are persisted so runtime doesn't re-derive them.
- **Daily feed, weekly team draft**: the lightweight daily feed and the optional richer weekly rollup
  are two different things, and nothing is posted without the user's OK.
- **Never fabricate**: every item comes from the pull or the paste; if a signal isn't in the data, say
  so rather than guess.
