---
name: digest-composer
description: 'Composes the structured outputs of producer skills (field calls, feedback and idea trends, support trends, engineering progress, competitor releases, adoption) into one short, readable daily digest. It narrates what the producers emitted this run and does not research on its own. It reconciles producers by shared subject, folds one story across every lens it touches, cross-links, right-sizes to one sentence per item, and leads with what matters most, at PM altitude (full per-lens detail) or PL altitude (cross-cutting leader synthesis). Single owner of cross-producer folding, the digest category model, density, and PM/PL altitude. Not for gathering signal (use the producer skills) or for scheduling, budget caps, or carryover (use daily-digest). Works on its own: with no producer outputs it offers to run whichever producer skills are installed, accepts pasted updates, and names what is missing. Trigger on "compose the daily digest", "what needs my attention", "my morning report", "run my digest".'
license: MIT. LICENSE has complete terms
compatibility: Standalone. No required connectors or skills. Composes whatever producer outputs it is given; if none, it runs installed producer skills (feedback-idea, field-qual, support-trends, engineering-progress, competitor-release) or works from pasted updates. The daily-digest skill, if installed, runs it last on a daily schedule. Needs no connector itself; producers bring their own (Pendo MCP and Listen, Jira, a call recorder, a support desk, a tracker).
metadata:
  author: Pendo Product Operations
  version: "1.1.0"
---

## When to use

Use whenever composing the daily digest, or when someone asks "what needs my attention" or for "my morning report." This is the daily digest composer: it narrates the outputs the producer skills emitted this run. It does not research on its own. It works for anyone on a product team: an exec gets awareness, a PM or PL gets the actionable detail.

# Digest composer

**This skill composes; it does not research.** Every source is owned by an independent **producer skill** that runs on its own cadence and hands back a short structured update. This skill takes the producer outputs from this run and narrates them into the **daily digest**: one glance, led by what matters most. (Research lives in the producers; this composer is the digest layer.)

## Done

- **Result:** one daily digest in chat: the header, the three static categories, any conditional
  categories that fired, the action surfaces that have items, and the set-aside line.
- **Done when:** each static category has an item or a calm or gap line; every link is a deep link
  to the specific artifact or a plain provenance reference; the digest is no longer than the heavy
  calibration example; the "set aside N" line is present.
- **Next consumer:** when the `daily-digest` conductor runs it, the conductor's editor pass, which
  applies the budget caps and carryover. Run alone, the person who asked.
- **Intent:** protect the reader's attention. When unsure, fewer honest threads over more.

## Boundaries and authority

Hard stops:

- Never open a source itself (a call recorder, Pendo, Jira, a support desk, a tracker), because the
  producers own their sources; name the gap and the producer that would fill it.
- Never merge on an account key alone, because account keys are often incomplete; relate the items
  instead.
- Never emit a stub or section-level link, because it lands the reader on a generic page; cite the
  provenance reference in plain text.
- Never present a staged item as created; render it as a 🟡 question until the user approves.
- Never render `internal_on_call_flags` as customer signal; it is a review-only list.
- Never carry a finding over from a prior run, because only this run's outputs are composed.
- Never fabricate a producer's section; an empty lens is a fact to state.

**Authority.** This skill writes nothing itself; its output is markdown in chat. In standalone
mode it may run installed producers, which make their own writes (for example
feedback-idea-producer's links and competitor-release-producer's config), and install missing
ones, each only after the user says yes.

## Judgment vs mechanics

The model judges: which items fold into one story (and the over-fold guardrail), ordering by
urgency times impact, the headline, light versus heavy day, PM versus PL altitude, and phrasing.
Producers compute and hand over the weights (linked ARR, SLA breach, tiers, concentration); this
skill does not re-derive them. These mechanics are still done by the model from prose and are
candidates for code: the provenance-key merge (non-account keys merge, account keys correlate
only within a namespace), support ordering by `tier` then `relevance`, and counting surfaced
items for the light or heavy call.

## Honesty

- An empty lens is valid: state it plainly and name the producer that would fill it.
- Couldn't check is not nothing there: a lens with partial coverage gets its caveat line and is
  never described as quiet.
- A producer's output is a claim; its link is the evidence. Never upgrade a paraphrase to a quote,
  an inference to a fact, or an unclassified item to relevant; label a possible duplicate rather than
  merge it.
- No time estimates beyond the read-length ceiling.

**Before presenting**, check: the static categories are present; one entry per issue; no inferred
why and no directive; every link deep or a provenance reference; length within the heavy example.
Fix what fails and check once more; if a check still fails, ship with a note on what failed.

## Input: what it receives

**Inside the conductor.** When the `daily-digest` conductor runs the day, it invokes each due producer, collects each one's structured output, and passes the set to this skill. It also passes the writer skills' results (for example feature validation, weekly updates) for the action surfaces.

**On its own (no conductor).** If this skill is invoked directly and no producer outputs are in the conversation:

1. Check which producer skills are installed (see the mapping table below). If any are, offer to run them now and compose from their outputs. Running a producer is not this skill researching; the producer owns its source.
2. Accept anything the user pastes (notes, a Jira export, a list of support themes, a feedback summary). Treat each pasted item as an item with **no provenance keys** (the soft-match rules below apply) and cite it as "from your notes" rather than inventing a link.
3. For each lens with nothing to compose, say so plainly and name the producer that would fill it. If it isn't installed, say it exists in https://github.com/pendo-io/Product-Loop and offer to install it. Never block on a missing producer.
4. If `daily-digest` isn't installed and the user wants this every morning, mention once that it runs the producers and this composer on a schedule.

In both cases this skill reads **only** the in-run outputs plus config (`digest.report_name`, `focus_areas`, `digest_budget`, `radar`, `comms_profile`, and **`role`**, which selects the altitude below). With no config: report name "Daily digest", role PM unless the user says otherwise (ask once if a human is present), and the budget defaults in the conductor's `config.example.json` (3 items per bucket, at most 3 proactive asks). It never opens a call recorder, Pendo, Jira, a support desk, or a tracker itself. If a producer didn't run or returned nothing, it has less to say and names the gap.

## The category model

**Static categories, always shown** (a single calm sentence when quiet):

- 💬 **From the field & feedback**: customer calls, feedback/idea trends, support trends, sentiment.
- 📈 **Adoption & usage**: recently-launched-feature adoption plus usage/analytics signal.
- 🚢 **Shipped & in-flight**: what shipped, moved, or is blocked (Jira), plus feature maintenance.

**Conditional categories, shown only when they fire** (no "nothing" line). On any day, light or heavy, a conditional category is hidden only when it has no item; 🧭 Competitive watch shows whenever it has one:

- 🧭 **Competitive watch**: relevant competitor releases in the user's areas. Appears just **below** From the field & feedback.
- ⚠️ **Conflicts**: a real collision (two efforts on the same problem/surface, a crossed boundary). Appears just **above** Heads-up and Shipped & in-flight; on a heavy day it may also drive the headline.
- ⚠️ **Heads-up**: something to watch that is not yet a collision or a blocker: a customer-facing regression not yet confirmed live, a post-ship quality readout, a cross-team dependency. Appears just **below** Conflicts and above Shipped & in-flight. It is a separate category from Conflicts: never merge the two headers or move an item between them.

**Action surfaces, beneath the signal, each omitted if empty:**

Feature validation sorts features into confidence tiers: Tier A it could corroborate and auto-validated; Tier B/C it is unsure about or believes are out of date, so it stages them for review.

- ✅ **Handled for you**: autonomous writer work (feature validation Tier-A, published updates), each a link plus a short "what changed."
- 🟡 **Needs your OK**: staged approvals (feature validation Tier-B/C, gap checks), each an interactive question plus deep link.
- 🔴 **Needs you to act**: only-you blockers (re-auth, a missing connector, engineering waiting on the user's answer). Rare; always shown when present.

## Producer → category mapping

| Producer output | Category |
|---|---|
| field-qual (cross-account call convergence), feedback-idea (idea trends + linked ARR), support-trends (support-ticket trends, account concentration, fade = fix confirmation) | 💬 From the field & feedback |
| competitor-release (`category: competitive_watch`: relevant competitor releases, patterns across competitors) | 🧭 Competitive watch (conditional) |
| launch-adoption monitor, usage / analytics trends *(no producer published yet; name the gap)* | 📈 Adoption & usage |
| engineering-progress (discovery deviations, shipped / moving / blocked, and product-consequential decisions from its `decisions` key: a scope cut, a descoped epic, a bug or story closed Won't Do, not planned or canceled, or a committed item moved to a later release) | 🚢 Shipped & in-flight |
| any producer flagging a collision / boundary crossing (including engineering-progress `conflicts`) | ⚠️ Conflicts (conditional) |
| engineering-progress `post_ship_regressions`, `cross_team_dependencies`, and `customer_facing_breaks` whose `why_it_matters` says it is a regression to watch | ⚠️ Heads-up (conditional) |
| an at-risk-account outage / SLA-breached escalation (support ↔ Jira, merged), engineering-progress `needs_you` (engineering is waiting on the user's answer), a `customer_facing_breaks` item that is live and hurting customers now, a re-auth / connector gap | 🔴 Needs you to act |
| writer skills' completed / staged / blocked results | ✅ / 🟡 / 🔴 surfaces |
| any producer's `staged_actions` (for example feedback-idea's idea links and `create_idea` drafts, support-trends' `jira_defect` or `content_ticket`, competitor-release's `triage_question` questions) | 🟡 Needs your OK, one question each with its link |

Producers hand over weighted signal (linked ARR, the annual recurring revenue of the accounts behind an idea or theme; SLA breach, account concentration, priority/severity, GA/launch context) that the composer uses to order (urgency×impact) and to lead the headline; it does not re-derive any of it.

## Reading producer fields

These rules apply on the consuming side only; they never change what a producer emits.

- **Support weight and staging.** Order support-trends `field_themes` by `tier`: `significant` before `mention` before `filler`, with `relevance` (`strong` > `moderate` > `weak`) as the tiebreaker. A theme with `staged: true` or `relevance: weak` goes to 🟡 Needs your OK, never into the lens body or the headline. A `filler` theme can appear in the lens but never drives the headline on its own; it leads only when it folds into a thread another producer corroborates.
- **Coverage caveats.** Read coverage from a producer's coverage field when it has one (support-trends `coverage`, field-qual `coverage_note`). Otherwise infer it from the producer's metadata (for example an item count equal to a known page or fetch cap, or a window narrower than the one asked for) or from its text (such as the coverage sentence at the end of feedback-idea's `insight`). When coverage was partial, add one short caveat line to that lens saying the pull was partial and what it missed. A quiet lens with partial coverage is never described as quiet; say the pull was limited instead. Omit the caveat when coverage was complete. There is no shared coverage key across producers.
- **Jira decisions.** engineering-progress `decisions` items (`key`, `summary`, `resolution`, `feature`, `goal_context`, `who`, `when`, `link`, `new_since_last_run`) are product-consequential decisions: a scope cut, a descoped epic, a bug or story closed Won't Do (or not planned, or canceled), or a committed item moved to a later release. Map them to 🚢 Shipped & in-flight at PM altitude, stated factually (what was decided, on which item, by whom, when), never editorialized. At PL altitude they are one of the leader-level inputs below, pitched at commitment/risk level without ticket jargon. Their `key` is the `jira:<KEY>` provenance, so a decision on an issue also reported elsewhere merges on that key.
- **Engineering heads-up and asks.** engineering-progress `post_ship_regressions` (`feature`, `since_ship`, `total`, `by_category`) render as one ⚠️ Heads-up line with the counts per category, never a list of bugs. `cross_team_dependencies` render in ⚠️ Heads-up; one with `composer_suppressible: true` is the first to go to the "set aside N" tally when the bucket is over budget. `needs_you` items (`key`, `summary`, `kind`, `who_waiting`, `link`) always go to 🔴, one line each naming who is waiting and on which item, linked on the item phrase; they are never suppressed or moved to another bucket.
- **Engineering-wait queue.** `support_trends.eng_wait_queue` = support-trends' `eng_waiting` entries with `queued_for_jira: true` and no `linked_jira`. engineering-progress reconciles those entries in `eng_wait_reconciled`; merge the pair on the shared `support_ticket` id (below). Link the merged item on the ticket phrase with the support entry's `link` (the ticket's own URL); when `link` is null or absent, cite `support_ticket:<id>` in plain text. `linked_jira: null` means there is no linked bug: render it as "no linked bug", never "bug link unknown" or "no bug filed". Keep each producer's severity: support's 🟡 for an escalated entry with no linked bug stays 🟡 when engineering-progress didn't run; when engineering-progress reconciles the entry, the pair is one 🔴 (next section).
- **Verbatim vs paraphrase.** Only text a producer marks as verbatim gets quotation marks (field-qual `representative_quote` in `topic_search` mode, a feedback-idea `quote` from the customer). A field-qual `account_scoped` quote is a paraphrase: render it without quotation marks and label it "paraphrased", attributed to the account and call. When unsure whether a quote is verbatim, treat it as a paraphrase.
- **New-idea drafts.** A feedback-idea `staged_actions` item with `kind: create_idea` carries a `draft` (`title` in customer words, `problem`, `who_affected` with `account_count` and `arr` where known, `quotes` with `link`s, `suggested_area`, `source_count`). Render it in 🟡 like any other staged action: one question naming the draft title and its `source_count` (distinct sources), with the draft's quote links, never as a created idea. Nothing is created until the user approves.
- **Competitor triage.** A competitor-release item with `triage: unclassified` is shown with that label, ordered after `triage: relevant` items, and never drives the headline on its own; its `triage_question` staged action is a 🟡 yes/no question ("Is this relevant to you?").
- **Cross-link types.** `cross_links[].related` values are hints for relating items by subject: `feedback_theme`, `support_theme`, `jira_item`, `adoption_signal`, `owned_feature`, and `competitor_release` (a Jira item or theme that bears on a competitor move). Any other value is ignored, never an error.
- **Field-qual extras.** `accounts[].deal` is the opportunity name and stage from a connected CRM, or null; name it as context when present, never infer one when null. `internal_on_call_flags` is a review-only list: never render it as customer signal, never count it toward a theme, and mention it at most as one 🟡 review line when the user asks for it. `link_scope` says what `moment_link` opens: `call` means link the call phrase ("[the Acme call](url)"), never imply it opens at the quote; `moment` may link the quote itself; null means no link, so cite `call:<id>` in plain text. `cross_source_hits` (`source`, `ref`, `account`) are non-call items that corroborate a call theme: mention them as corroboration only, and never count them toward convergence or toward a theme's account count, which come from calls alone.
- **Unknown keys.** A producer may send a key that is not in its known contract. If it reads as human-facing (it carries a summary or one-line sentence, ideally with a link), pass it through into that producer's category as a low-priority item, ordered after the known items and never in the headline. Otherwise ignore it and count it in the "set aside N" line. Never error or skip the producer because of an unknown key.

New producers slot in here by declaring which category they feed, with no change to this skill's logic. Producers are matched by the role they play, so a producer published under a different name (for example a support-trends producer from another catalog) maps the same way.

## Reconcile & fold: the composer's core job

Each producer sees one source; **the same real story usually shows up in several of them, and folding it into one thread is the main value this skill adds.** Before composing, reconcile across producers by **shared subject** (account, feature, theme), not by producer:

- **Match on provenance keys first, then subject.** Every producer stamps each item with a `provenance` set of stable keys: `feedback_item:<id>`, `idea:<id>` (feedback-idea, from the Pendo MCP), `support_ticket:<id>` (`support-trends-producer`, and field-qual's cross-source items), `jira:<KEY>` (engineering-progress, holding any tracker's issue ref, and any support ticket's linked Jira), `call:<id>` (field-qual), `competitor_release:<id>` (competitor-release), and a **type-qualified** account key: `account:crm:<id>`, `account:analytics:<id>`, or `account:domain:<x>` (any producer), plus `account:name:<name>` as a fallback when a producer has no CRM id or domain (field-qual emits it when the call tool returns neither). Two items whose provenance sets share any **non-account** key are the **same underlying artifact: merge them outright.** This is exact, not a judgment call: the same support ticket surfaced by `support-trends-producer` *and* as one of field-qual's cross-source items, or a Jira issue reported by engineering-progress *and* linked from a support ticket, collapse to one. A `competitor_release` key shares no artifact with any other producer, so it only de-dups that producer and anchors cross-links by subject; it never hard-merges. The account key is a **correlator, never a hard merge on its own**: the same account across lenses justifies *relating* findings, not collapsing them. Match account keys **only within the same namespace** (`crm:` to `crm:`, etc.). `account:name:` keys match only other `account:name:` keys, and only loosely (same name after trimming case and punctuation): enough to relate items and fold a story by subject, never to merge. An account key in any other unrecognized namespace is treated the same way, as a label for soft-matching only. To bridge namespaces, use a CRM crosswalk when one exists: if a product-analytics account record (for example a Pendo account) carries its CRM account id (for example a Salesforce id) in its metadata, feedback-idea can emit **both** `account:analytics:<id>` and `account:crm:<id>`, letting its items correlate with field-qual's CRM-keyed accounts. **A support desk usually has no shared account id**, so support account correlation is `account:domain:<x>` only: matched loosely and labeled, never merged. Account keys are often missing from some records, which is another reason they never hard-merge. Items that carry **no key** (email / NPS / Slack mentions, pasted notes, and theme-level rollups with no contributing ids) fall to the subject rules below: soft-match on same account + same subject within the window and **label it a possible duplicate; never silently merge** (the over-fold guardrail governs). Provenance-key matching is the exact first pass; the subject-fold rules below are the semantic layer on top. **Keyed items fold on similarity too:** two keyed items that share no key can still fold into one thread when they describe the same issue, root cause, or customer pain (same account or feature and the same subject); a shared key is not required for a fold. Only a shared non-account key merges outright; a similarity fold keeps each item's own link, and when two keyed items look like the same artifact but share no key, label them a possible duplicate rather than merging.
- **Fold one story across every lens it touches into a single thread**: a *single* customer pain that appears on calls (💬) and in support volume (💬) is **one** thread with reciprocal cross-links, never separate bullets. (Example: a resource manager saying on a call that people on holiday still show as free, plus a support theme about capacity shown for staff on leave, is one customer story: the planning view ignores time off. It leads.)
- **State what happened, not an inferred *why*.** Relay the fact a producer gives you; never add a reason, motive, or judgment the source didn't establish. *"The team moved the template import to next sprint"* is a fact; *"…because review ran late"* is an inference, so drop it unless a producer actually reported it. Inferring intent is the same failure as over-folding: it manufactures a claim the data doesn't support. **And don't prescribe what the reader should do, or assign who should act.** *"Yours to unblock,"* *"reach out to the account owner,"* *"you should escalate"* all put words in their mouth. Surface the situation and its link; the reader decides what to do with their own attention. (A staged skill action offered as an option, a `[ Open task ]` / `[ File bug ]` affordance, is fine; a directive about how the *person* should spend their time is not.)
- **Don't over-fold (guardrail).** Fold only on a **genuinely shared issue, root cause, or single customer pain**, never a merely shared *topic*; "both are about data" is not a fold. And **do not pull an internally-shipped item into a customer thread unless customers actually reported it**. For example, a refactor of how reminder emails are queued is internal work *no customer raised*, so it stands on its own in 🚢; cross-linking it to a customer thread about planning because both touch notifications would manufacture a false "one story," which is worse than two honest ones.
- **Merge producer hand-offs into one item.** When `support-trends-producer` flags a waiting-on-engineering escalation with no linked bug (`queued_for_jira: true`, so it is in `eng_wait_queue`), and engineering-progress reports that same ticket in `eng_wait_reconciled`, surface **one 🔴**, not two. Deep-link it on the ticket phrase with the ticket's `link` (support-trends' `eng_waiting[].link`, which engineering-progress carries through on `eng_wait_reconciled`); when it is null, cite `support_ticket:<id>` in plain text. A support fade that confirms a shipped fix (tickets about calendar double-bookings stopped once the sync retry change went out) is **one** closed-loop note.
- **Correlations worth drawing** (raise weight when they co-occur): a support/Jira bug ↔ the feature it hits; a support theme ↔ the matching call theme ↔ an existing feedback idea (same pain, multiple channels); **slow or at-risk support × negative customer sentiment on the same area** (compounding, so lift it); a fade/fix in one source confirming a fix reported in another; a competitor release ↔ the owned feature or feedback theme it bears on.
- **De-dup by subject, then tally.** One entry per real story; count the rest in the "set aside N" line.

## PM vs PL: same inputs, different altitude

- **PM (`config.role` = `PM` or `Other`; owns specific features):** every lens in full, using the categories as written and the folded threads above. (`Exec` gets a separate cross-area roll-up from the conductor, not this branch. If this skill is run standalone for an `Exec`, compose at PL altitude and say so.)
- **PL (owns a portfolio of areas / has PMs under them; `config.role` = `PL`):** a PL does not read per-area producer dumps. Compose the **cross-cutting, leader-level** picture only: portfolio-spanning conflicts, **commitments / launches at risk**, escalations on **at-risk accounts**, and **product-consequential decisions** (engineering-progress `decisions`), each synthesized *across* their areas. Producers feed this as **corroboration**; a single producer item stands alone **only when it's leader-critical and uncorroborated** (for example a launch commitment slipping with nothing else flagging it). No "every support theme / every Jira ticket" list; the lenses still frame the read, but each is a synthesis, not a per-area enumeration. **Pitch it at commitment/risk altitude, and strip PM/ticket jargon**: a PL reads *"a committed launch is shipping with unresolved issues,"* not *"the rollout flag is still at 20 percent"* / *"a P2 regression was reopened"* / *"the `capacity_hours` column double-counts part-time staff."* Ticket types, severities, field names, and internal gate names belong in the linked source for drill-down, never in the PL's line. If you can't say why an item matters to someone who owns the *portfolio* (not the tickets), it doesn't belong in the PL digest.

## Adaptive density: headers never change, only detail

- **Light day** (fewer than 8 surfaced items after folding, and no dominant story): one readable sentence per item, calm, no judgment headline. Conditional categories still show when they have an item (a relevant competitor release still gets its 🧭 Competitive watch line) and are hidden only when empty. A quick skim.
- **Heavy day** (8 or more surfaced items after folding, **or** one story that dominates the day, for example a 🔴 item touching several lenses): open with a short **judgment headline** naming the one thing that matters most (which may be a conflict), then fuller detail per category. With 8 or more items, any item may run to **two sentences**. When the day is heavy only because of a dominant story (fewer than 8 items), that story leads and may run to **two sentences**, and every other item stays at one. Still a ≤3-minute read.

Judge the load; never force a headline on a calm day, never bury a real story on a busy one.

## Voice & rules

- **Right-sized, readable sentences**: **one complete, meaningful sentence per item**. Say *why* it matters, but not a paragraph and not a telegraphic fragment. Enforce this here so a verbose model doesn't balloon and terseness doesn't gut the meaning; it must hold in a plain setup, never relying on the reader's own concision settings. (A folded thread may run to two sentences when it spans lenses; on a heavy day of 8 or more items, any item may run to two sentences; on a day made heavy by one dominant story, that story may run to two sentences; otherwise a single item is one.) Use the `comms_profile` voice if one is set.
- **≤3-minute read**, heavy day included; a light day is a ~30-second skim. A digest longer than the heavy calibration example below is a bug.
- **One issue, one entry**: fold a topic's signal + escalation + action together; never fracture it. **Cross-link** a shared story across categories (customer pain in field ↔ the fix in shipped), each pointing to the other.
- **Discovery, not recap**: producers surface only what's *new since last run* and exclude what came straight to the user (their DMs/emails/own calls); when several already-seen items form an escalating trend, surface the *trend* as a heads-up. Close with a one-line "set aside N already-seen items" tally.
- **Link inline, on the phrase you name.** Wherever you mention a specific thing, hyperlink the **phrase itself** to its source; don't relegate links to a trailing arrow. A single item links to its own URL (a Jira issue, a support ticket, a recorded call, a feedback idea, a competitor release); a **cluster/theme** links to a **query/view spanning its items** when one exists (a JQL search, a saved support view), else to the clearest single example. Producers hand you the URLs (jira `link`/key, support `support_view_link` and each `eng_waiting` entry's `link`, feedback `idea_link`, call `moment_link`, competitor `url`, and for a cluster the query URL), so place them on the noun phrase (*"[weekend hours](url) were descoped"*, *"the [time-off idea](url) gained four items"*, *"the [leave-capacity tickets](view-url) that rose this week"*). Keep a single trailing affordance per action item only for the interactive approval (the `[ Review 2 ]` question, or the resolve-here link). **Every link must be COMPLETE: a deep link to the specific artifact, never a bare section/app URL.** An app's home page or a call recorder's root URL is a stub, not a link; the real target is the specific idea, call, issue, or ticket page. If a producer didn't hand you a complete URL for a thing, **cite it in plain text by its provenance reference (`call:<id>`, `jira:<KEY>`); never emit a truncated or section-level href that drops the reader on a generic page.** A link that doesn't land on the exact item is a defect.
- **Ordering** = categories in the order above (field & feedback first, since customer signal leads); within a category, order by urgency×impact.

## Calibration examples: match these

Both are the **heavy/rich** end (8 or more items after folding, with a real cross-cutting story), so an item may run to two sentences. A light day (fewer than 8 items and no dominant story) is one calm sentence per item, no headline, and conditional categories only when they have an item. Accounts, areas, and numbers are illustrative.

### PM (owns specific features): full detail, folded threads
```
🔭 Daily digest · Workload View · Thursday
The story worth your attention is time off: Workload View shows people on leave as free, and that
now turns up on calls, in feedback and in support in the same week a competitor shipped leave-aware capacity.

💬 From the field & feedback
 • Time off is the through-line: on [the Globex call](call-url) a resource manager said they keep
   booking people who are on holiday, the [show time off on the grid idea](idea-url) gained four items
   from three accounts, and [capacity shown for people on leave](support-view-url) is a new support theme.
 • Acme [praised drag-to-reassign in feedback](feedback-url): "it's the quickest way we've found
   to rebalance a week."
 • Support coverage was partial this run: the pull stopped at the desk's page limit, so the support
   trend reflects only part of the window.

🧭 Competitive watch
 • [Competitor A added leave-aware capacity to its planner](release-url), which bears directly on
   the time-off thread above.
 • [Competitor B shipped task-template import](release-url), unclassified until you answer below.

📈 Adoption & usage
 • Nothing to read here: no adoption producer ran for Workload View this morning.

⚠️ Conflicts
 • Two epics change how available hours are calculated: Workload View's
   [part-time capacity](jira-url) and Plan & Track's [sprint capacity rework](jira-url).

⚠️ Heads-up
 • Since [bulk reassign](jira-url) shipped, three new bugs came in: two about undo, one about notifications.

🚢 Shipped & in-flight
 • [Bulk reassign](jira-url) shipped to all workspaces, the last open item on the rebalancing goal.
 • Sam Okafor descoped [weekend hours in capacity totals](jira-url) from the part-time epic on Monday.
 • A [Calendar Sync timezone bug](jira-url) was reopened after QA hit it across a clock change; it
   shares a connector with the time-off thread but not its cause, so it stands on its own.

🟡 Needs your OK
 • Connect the time-off idea to its [Workload View feature record](feature-url)?  [ Connect ]
 • Is [Competitor B's template import](release-url) relevant to you?  [ Yes ] [ No ]
 • Feature validation found the [Workload View feature record](feature-url) marked GA while its
   [epic](jira-url) is still open; review the staged edit?  [ Review ]

🔴 Needs you to act
 • Engineering is waiting on your answer to [whether weekend hours count for part-time staff](jira-url)
   before the part-time epic can continue.

Set aside 12 items already seen or out of area.
```

### PL (owns a portfolio): evidence mode, cross-cutting only, no per-area dumps
```
🔭 Daily digest · Portfolio (4 areas) · Thursday
Two things rise to your level: Workload View and Plan & Track are both changing how available hours
are calculated, and time off is becoming a cross-area customer theme that a competitor now addresses.

💬 From the field & feedback
 • Customers on calls, in feedback and in support say planning ignores time off; it starts in
   Workload View and depends on Calendar Sync, so it spans two of your areas.

🧭 Competitive watch
 • [Competitor A now offers leave-aware capacity](release-url), the same gap your customers are naming.

📈 Adoption & usage
 • No adoption read: no adoption producer ran for these areas.

⚠️ Conflicts
 • Two teams are reworking the same capacity calculation in parallel
   ([part-time capacity](jira-url), [sprint capacity rework](jira-url)), which puts both releases at risk of disagreeing.

🚢 Shipped & in-flight
 • The part-time capacity release was narrowed: [weekend hours](jira-url) are out of its first version.

Per-area support themes, individual tickets, and routine shipping rolled up to your PMs;
surfaced here only what needs a leader. (No ticket jargon, no directives: the situation and its link; the read is yours.)
```

## Gotchas

- **Never fabricate a producer's section.** If a producer didn't run or returned nothing, say so plainly ("Adoption is dark: no usage producer connected for these areas") and move on. An empty lens is a fact to state, never content to invent.
- **Only narrate producers that actually ran this turn** (or updates the user pasted this turn). Don't reference a source the user hasn't wired, and don't carry a finding over from a prior run's context; compose from this run's outputs.
- **Density-adapt, don't pad.** A quiet day is a short digest; a busy day (8 or more items, or one dominant story such as a 🔴 item touching several lenses) leads hard and still right-sizes. Every surfaced item is one complete, meaningful sentence (up to two on a heavy day of 8 or more items; on a day made heavy only by a dominant story, only that story gets two), never a paragraph write-up, never a cryptic fragment.
- **Fold on a shared issue/root-cause, not a shared topic** (over-fold guardrail), and **state what happened, not an inferred why**; don't prescribe what the reader should do or assign who should act.
- **PL altitude strips ticket jargon**: commitment/risk level, with specifics left in the linked source.
- **Every claim carries its source link.** If you can't link it, you probably can't support it. Pasted items with no link are cited as "from your notes."

## Conductor notes

- **Runs last** in the conductor's daily run, after the producers and writers, so it has their outputs to compose. Not separately scheduled.
- **Non-interactive** on scheduled runs; it states, and the 🟡 questions are handled by the conductor's editor pass. When run on demand with a human present, render 🟡 items as interactive questions if the client supports them (for example Cowork's ask-user tool), else as plain questions with their links.
- **Degrade gracefully**: compose whatever producers ran; name a producer that failed or wasn't connected rather than inventing its section.
- **Producers are pluggable**: a new producer just declares its category and cadence; this skill needs no change to absorb it.

## Principles

- **Compose, don't research.** All signal comes from producer skills (or what the user pastes); this skill is the editor.
- **Static categories, conditional Competitive watch, Conflicts and Heads-up, adaptive density.** Headers are stable; detail flexes with load.
- **Readable, not choppy**: full sentences, ≤3-minute ceiling.
- **Discovery, not recap; customer signal leads.**
- **Fold across producers by subject**: one real story that shows up on calls, in support, and in Jira is one cross-linked thread, never repeated per lens; merge producer hand-offs (a support escalation + its Jira bug) into one item.
- **PM vs PL altitude**: a PM gets full per-lens detail; a PL gets the cross-cutting leader read (commitments/launches at risk, at-risk-account escalations, portfolio conflicts), producers as corroboration, no per-area dumps.
- **Every action deep-linked; nothing that needs the user is dropped.**
