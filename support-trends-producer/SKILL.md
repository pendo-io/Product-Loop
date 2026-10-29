---
name: support-trends-producer
description: 'Produces a daily digest of support-ticket trends for a PM. Reads recent tickets from any connected support tool (Zendesk, Intercom, or similar) or a pasted/CSV export, classifies them to the PM''s product areas by area names and aliases, and surfaces the trends forming across those areas (recurring themes whose volume deviates from its baseline) that the PM might have missed. Flags tickets waiting on engineering with no linked bug. Renders a digest alone, or a delta-only structured update for a digest composer. Single owner of support-ticket trend detection and of the eng_wait_queue definition. Not for bug status (use engineering-progress-producer), feedback themes (use feedback-idea-producer), or composing the digest (use digest-composer). Read-only on the support tool, tracker and feature tracker; writes only its own config, plus a skill install on your OK. Runs on demand or daily. Triggers: "support trends", "zendesk trends", "what''s hitting support", "ticket themes in my area", "support signal".'
license: MIT. LICENSE has complete terms
compatibility: Standalone. No required connectors or skills; works from a pasted or CSV ticket export. Optional connectors - a support tool (Zendesk, Intercom, Freshdesk, or similar), Jira (linked bugs), a CRM (account revenue and health), a feature tracker such as Notion (scope anchor), Pendo Listen via Pendo MCP (matching feedback ideas). Optional companion skills - digest-composer, engineering-progress-producer, feedback-idea-producer.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

## When to use

Use for a PM's support-ticket trends across their own product areas: recurring themes whose volume is deviating from baseline. Triggers: "support trends", "zendesk trends", "what's hitting support", "ticket themes in my area", "support signal". Surfaces trends, not individual tickets, writes only its own config plus a companion skill install on your OK (see Boundaries and authority), and runs on demand or daily.

# Support Trends Producer

A **producer**: it owns one signal (support tickets, classified to the user's product areas) and produces a short, structured, **delta-only** update each run. It works two ways:

- **Run alone** (the default): it renders its own readable digest in chat (see "Standalone digest" below). Nothing else needs to be installed.
- **Inside a daily digest:** when a digest composer such as `digest-composer`, or an orchestrator that runs producers daily, is present, it hands over the structured block below. It adds **no section of its own**: the composer folds its output into the daily digest's existing 💬 From the field & feedback category. Both skills are available at https://github.com/pendo-io/Product-Loop. A setup from before the composer's 1.1.0 rename may have the full composer under its old name; that copy counts as an installed composer (a deprecated stub, whose description starts with "Deprecated", does not), and a missing composer is offered as `digest-composer`.

Its job: answer *"what support-ticket trends are forming across my areas that I might have missed, and which themes are spiking or fading against their normal volume?"* as a **trend picture**, never a ticket dump. Whether those trends align or collide with other signals (a customer call, a feedback theme) is the composer's call, not this skill's. This producer emits its own signal and the raw account facts; the composer does the cross-signal correlation.

## Done

- **Result:** a support-trends digest in chat (Standalone digest), or, under a composer, the
  structured block in step 6.
- **Done when:** every surfaced theme shows current vs baseline and the denominator; the Coverage
  line is present with each area's tiers and their basis; a 🔴 appears only on a known SLA breach
  with no linked bug and no escalation tag. A quiet run is the calm line with the coverage note.
  When `eng_wait_statuses` is unset or still the placeholder, the coverage note says stuck
  escalations were not checked.
- **Next consumer:** `digest-composer` or the `daily-digest` conductor (💬 From the field & feedback),
  and `engineering-progress-producer`, which reads `support_trends.eng_wait_queue`. Run alone, the
  person who asked.
- **Intent:** a trend picture, never a ticket dump. When steps run out, name the coverage gap
  rather than read a thin pull as quiet.

## Boundaries and authority

Hard stops:

- Never quote a customer from a support ticket, because support language is often heated and the
  issue is already being worked; state the theme neutrally from the agent's read.
- Never claim "no new support signal" off a thin or partial pull, because a change in coverage is
  not a change in support; say coverage is limited.
- Never raise a 🔴 without a known SLA breach, no linked bug, and no escalation tag, because 🔴
  needs a known breach; stage it as 🟡 instead.
- Never guess an account name, ARR, or health; label the account by its id and leave CRM fields
  `null`.
- Never prompt on a scheduled run; use config, and if a required piece is missing, emit nothing and
  name the gap.
- Never create a new feedback idea; offer to link the theme to an existing one.
- Never report a config write as saved without reading it back, because a failed write would
  silently lose the cache or the queue; re-read `config.json` after each write and check the
  changed keys, and confirm an installed companion's `SKILL.md` is on disk (for the composer,
  `digest-composer` or the full composer under its pre-1.1.0 name).

**Authority.** Every write this skill can make:

- Its own config (`support_trends` namespace, in `config.json` next to the skill or in the
  conductor's shared config): first-run answers when the user agrees to save them, `area_map`
  aliases the user confirms, `sla_ref` once confirmed, the `area_history` cache on each history
  refresh (also on scheduled runs), and `eng_wait_queue` each run.
- A skill install, only on the user's OK: downloading a missing companion skill's folder into
  `~/.claude/skills/<skill-name>/`.
- Staged only: content tickets, bug filing, and idea links are 🟡 actions carried out by another
  skill or by hand after approval.

Read-only on the support tool, issue tracker, CRM, feature tracker, and feedback tool.

## Judgment vs mechanics

The model judges: area classification and relevance tier, the shared-area narrowing, theme
clustering, concentrated versus spread, the low-volume escape hatch, and phrasing. Connectors
gather tickets, organization names, linked bugs, and account facts. These mechanics are still done
by the model from prose and are candidates for code: dedupe on `(source, ticket_id)`, the per-area
weekly median, the tier formula, window deltas normalized to the denominator, and SLA wait and
breach status.

## Honesty

- An empty result is valid: one calm line with the coverage note, never padded.
- Couldn't check is not nothing there: a thin pull, a missing history, or missing export columns
  are named in the coverage note; "no linked bug" claims only that no bug is linked.
- An unset waiting-on-engineering status is not an empty queue. When `eng_wait_statuses` is empty
  or still holds the placeholder (any value in angle brackets), skip step 5b, put "waiting-on-engineering
  status not set; stuck escalations not checked" in the coverage note, and never report "no stuck
  escalations".
- Label proxies and small samples: tiers from floors or a partial history are named as such; a
  low-volume single is framed as a single, never a trend.
- No time estimates.

**Before presenting**, check: no thin pull read as quiet; no ticket dump; no customer quote; no
🔴 without a known breach; no guessed account name. Fix what fails and check once more; if a check
still fails, ship with the failure named.

## Sources

Tickets come from whichever of these is available, in this order:

1. **A connected support tool** (Zendesk, Intercom, Freshdesk, Help Scout, or similar). Use its search/list tools for tickets updated in the window, and its get-ticket tool for the waiting-on-engineering detail.
2. **A pasted or uploaded export** (CSV, JSON, or a table). Useful columns: ticket id, created/updated date, subject, the latest human-authored agent comment or internal note, tags, status, priority, organization/account, linked bug key/URL, and ticket URL. Work with whatever columns exist and say which were missing.

Optional enrichments, used only if connected: **Jira** (or another issue tracker) for linked-bug status; a **CRM** for account revenue, health, and renewal date; a **feature tracker** (a Notion database, Jira, Airtable, or similar) listing the user's features and the product areas they belong to; **Pendo Listen** via Pendo MCP (or another feedback tool) for matching existing ideas.

> **Contract note.** The config namespace and output keys (`support_trends`, `support_ticket:<id>`, `support_view_link`, `crm_account_id`) are stable, tool-neutral contract keys that other skills read. They apply to tickets from any support tool (for example Zendesk) and any CRM.

## Config it reads (never hardcode identity)

All under `support_trends` unless noted. See `config.example.json` for the shape.

- `focus_areas` (top level): the user's product area(s). If a feature tracker is connected, the user's own features and the areas they link to are the authority; otherwise `focus_areas` is.
- `support_trends.area_map`: per area, the names, aliases, support-tool tags or tag prefixes, and (optionally) feature names used to classify a ticket to that area. The skill classifies tickets itself (see "Classify to areas"). Seed it from the user's answers on first run and from `references/area-map.example.json`; add aliases as they're confirmed. Silent on scheduled runs.
- `support_trends.routing_rules`: optional overrides applied before bucketing, for example "tickets tagged `calendar-sync` belong to *Calendar Sync* even when the text mentions *Plan & Track*," or "exclude tags starting with `billing-`." Default: none.
- `support_trends.drop_if_no_organization`: drop tickets with no organization/account (usually spam). Default `true`.
- `support_trends.features_db`: optional feature-tracker location (database or project URL). Used to resolve the user's areas and to narrow an area the user shares with other PMs.
- `support_trends.stage_only_areas`: areas whose classification is not yet trustworthy enough to auto-surface; their items are **staged (🟡), never auto-surfaced**. Default `[]`. Add an area when its matches keep picking up the wrong tickets (for example a newly added area with no aliases or tags yet, whose name-only matches pull in unrelated tickets).
- `support_trends.baseline_window_days`: rolling window for trend detection. **Default 7.**
- `support_trends.thresholds`: the absolute deviation tiers, counted in tickets (not comments), with defaults `{ big_deal: 12, mention: 5 }`. These are the **caps**: each area's own tiers (below) never exceed them, so high-volume areas behave as before.
- `support_trends.history_weeks`: how many weeks of ticket history set each area's typical weekly volume. **Default 8.**
- `support_trends.area_tiers`: how each area's tiers are derived from its own history. Default `{ mention_multiplier: 0.5, big_deal_multiplier: 1.0, mention_floor: 2, big_deal_floor: 4 }`. See "Per-area tiers" under step 5.
- `support_trends.area_history`: the skill's own cache of each area's typical weekly volume, written by the history refresh. Shape `{ refreshed_at, refresh_days, source, areas: { "<area>": { typical_weekly, weeks_observed } } }`. `refresh_days` default **7** (refresh weekly). Starts empty; never hand-edited.
- `support_trends.eng_wait_statuses`: the ticket status(es) that mean "waiting on engineering." Set it to the exact status name(s) your support tool uses; the example config holds the placeholder `["<your 'waiting on engineering' status>"]`, and setup asks for it. Until it is set, step 5b is skipped and the coverage note says so (see Honesty). Matched against the ticket's custom status label when the tool has one (in Zendesk the label lives in `custom_status` while `status` reads `hold`), its standard status, and its tags (a tag such as `<your waiting-on-engineering tag>` that normalizes to a listed value).
- `support_trends.eng_wait_queue`: **defined as this producer's `eng_waiting` entries with `queued_for_jira: true` and no `linked_jira`** (tickets waiting on engineering with **no linked bug**). Written from `eng_waiting` each run and handed to `engineering-progress-producer` to find or file the bug (self-maintaining; entries clear when a bug link appears). The same definition is used by `engineering-progress-producer`, `digest-composer`, and the conductor. See the handoff below.
- **Org-wide values.** When the daily-digest conductor runs this skill (or its shared config exists at `<workspace>/.daily-digest/config.json`), read `org_constants` there first for org-wide values, then this skill's own settings. This skill reads `org_constants.sla_reference` (for `sla_ref`) and `org_constants.support_desk_domain` (for ticket links). With neither present, it uses the keys below unchanged.
- `support_trends.sla_ref`: the user's own priority and severity targets. Read `org_constants.sla_reference` first: if it holds the targets as text, read them into this shape; if it holds only a link, use the targets saved here after the user confirmed them from that page once (attended); with nothing usable there, use this key. Shape, for example: `{ "priority_targets": { "High": "3d" }, "resolution_slas": { "P1": "2 business days" } }`. No default values: if it isn't set, compute wait time only and emit `sla_status: null`.
- `support_trends.ticket_url_template`: how to build a ticket link from its id, for example `https://<your-subdomain>.zendesk.com/agent/tickets/{id}`. Read `org_constants.support_desk_domain` first: when set and the tool is Zendesk, build `https://<support_desk_domain>/agent/tickets/{id}`; for another tool, put the domain in this template's host. If neither is set and the source gives no URL, cite the ticket by id in plain text.

On a **scheduled run never prompt**: use config; if a required piece is missing, emit nothing and name the gap.

Every run rebuilds state from config and the ticket source, never from earlier conversation.

## First run (attended, standalone)

If there's no saved config, ask in one short message, then offer to save the answers as a `config.json` next to the skill (or, when the daily-digest conductor's shared config exists at `<workspace>/.daily-digest/config.json`, there):

1. Your product area(s), and any other names, tags, or feature names support uses for them.
2. Where tickets come from: the connected support tool, or paste/upload an export now.
3. Your bug SLAs: the target for each priority or severity your team uses (for example High within 3 days, P1 within two business days). These set `sla_ref`, and only a breach of one of them can raise a 🔴. If the shared config's `org_constants.sla_reference` is set, confirm it instead of asking again. If the user has none, say that stuck escalations will stay 🟡.
4. Optional: a feature tracker to anchor scope, the status that means "waiting on engineering," and the ticket URL pattern. Without the status, stuck escalations aren't checked, and every run's coverage note says so.

Defaults apply to anything skipped. Don't ask again once saved.

Once areas and the ticket source are set, run the **history refresh** (step 2b) so each area starts with its own tiers, and save the result to `support_trends.area_history`. With a pasted or CSV export, refresh only if the export spans at least two full weeks; otherwise leave the cache empty and use the floors.

## Scope: the user's areas, feature-narrowed for shared areas

1. Resolve the user's areas: from their features in the feature tracker (each feature's linked product area) if one is connected, else `focus_areas`.
2. Classify tickets to those areas (below). Classification is **area-grain**, so an area the user shares with other PMs returns the whole area's tickets.
3. **Narrow shared areas to this user's features.** When an area has several owners in the tracker, do a lightweight semantic match of each ticket's text to **this** user's features in that area, and drop the ones that clearly belong to a co-owner's feature. When the user owns the whole area, or there's no tracker, skip this pass.

## Classify to areas

For each ticket, after the hard filters (`routing_rules` exclusions, then `drop_if_no_organization`):

**Classification is text-first.** Support-tool search results often show only some of a ticket's tags (Zendesk search can truncate the list and report the full `tag_count`), so tags from a search result are partial. Classify on the subject and human-authored comment first, using whatever tags are visible. Fetch the full ticket (the get-ticket tool) only for **borderline** tickets, where the text match is loose or the visible tags are clearly truncated (`tag_count` above the tags shown) and the full tags could change the tier. Don't fetch every ticket. From an export, use the tags column as given.

- Apply `routing_rules` overrides first.
- Match the subject plus human-authored comment against the area's names, aliases, and feature names, and the ticket's tags (visible, or full when fetched) against each area's tags/prefixes.
- Assign a **relevance tier**: **strong** when a tag match and the text agree, **moderate** when only one of them matches clearly, **weak** when it's a loose text match only. A ticket may match more than one area; keep every match but count it once.
- Tickets that match none of the user's areas are out of scope (count them in `set_aside_count` only if they were plausibly close).
- When the user confirms or corrects a match in an attended run, add the alias to `area_map`.

## Data reality

- **Coverage is what was actually pulled and classified.** The denominator (`evaluated_items`) is the number of tickets pulled for the window and run through classification. A connector page limit, a truncated export, or a short export window shrinks it. Phrase every volume or trend claim against that denominator, and never report "no new support signal" off a thin or partial pull; say coverage is limited.
- **Ticket grain.** Count tickets, not comments. **Dedupe on `(source, ticket_id)`**, and because a ticket can match more than one area, never count it once per area.
- **Route by area + volume + account health, not by the relevance tier.** Use the tier only as a tiebreaker, and to stage weak-tier matches. `stage_only_areas` items are **always staged (🟡)**, regardless.
- **Account facts.** Use the ticket's organization/account. Search results may carry only an `organization_id` with no name: resolve names with the support tool's organization lookup (for example Zendesk's get-organization tool), once per distinct id, for the accounts you will name (concentration, `top_accounts`, `account_signals`). If no lookup is available or it fails, count distinct accounts by id and label the account by its id (for example "org 12345"); never guess a name. If a CRM is connected (or the support tool carries it), add revenue and health: `active_arr`, `next_renewal_date`, `days_to_renewal`, `health_label` (At-Risk / Stable). If not available, leave those fields `null` (and `enterprise_atrisk: false`), say so in `coverage.note`, and never guess them. Account bridge key: `account:crm:<id>` when a CRM account id (for example a Salesforce id) is present, else `account:domain:<x>` from the requester's email domain, else the account name; matched loosely, never a hard merge.
- **Waiting-on-engineering detail.** For the escalation candidates only, read the ticket's status (including its custom status label), tags, linked bug (key and URL), and priority. Keep this targeted: a handful of flagged tickets, not the broad pull. From an export, use the status and linked-bug columns.
- **No customer quote from a support ticket.** State the theme neutrally, grounded in the support agent's read (the human-authored comment), never a raw customer quote. Support language is often heated and the issue is already being worked. (Deliberate difference from the feedback and field-call producers.)

## Daily workflow

### 1. Resolve scope
Resolve the user's areas and `area_map` (above). If a user area has no aliases or tags yet, classify it by name only and note that in the run notes ("area X has no aliases configured, matches may be thin").

### 2. Size it: per-area volume + denominator
Pull tickets for the current window and the prior window of the same length (`baseline_window_days`). Classify both. Current-window vs prior-window counts per area give the cheap baseline, with `evaluated_items` as the denominator.

### 2b. History refresh (slow cadence, cached)
Run this at first setup, when `area_history.refreshed_at` is older than `refresh_days`, and for any area added to `area_map` since the last refresh. Otherwise read the cache; don't re-pull history every run. It is a read plus a write to the skill's own config, so it also runs on scheduled runs without prompting.

1. Query the connected support tool for tickets over the last `history_weeks` weeks (default 8), paging to completion, with the same hard filters, dedupe, and classification as the normal run (Classify to areas).
2. Count in-area tickets per area per calendar week, including weeks with zero tickets.
3. Store each area's **typical weekly volume** as the median of those weekly counts, with `weeks_observed`, `refreshed_at`, and `source`, in `support_trends.area_history`.

**No history available:** a pasted or CSV export covering a single window (under two full weeks), a support tool that can't return that range, or an area added since the last refresh with no way to refresh now. Don't store a value for that area; its tiers fall to the floors (step 5) and the coverage note says so. A history pull that hit a page limit or returned partial weeks is stored with its `weeks_observed` and named as partial in the coverage note.

### 3. Pull the items (complete, deduped)
Page through the support tool's results until no pages remain; never conclude from a partial first page. Read the comment text for the tickets you'll theme. Dedupe on `(source, ticket_id)`. For shared areas, run the narrowing pass (Scope, step 3).

### 4. Cluster into themes
Roll the in-scope tickets into recurring **themes** per area (and, where derivable from the text, per feature). **The theme is the unit; never surface individual tickets**, except the two named cases: a waiting-on-engineering ticket (step 5b) and a low-volume single (step 5). Build the theme statement from the support agent's assessment (the human-authored comment), stated neutrally; **no customer quote.**

**Characterize each theme's account concentration; the same volume means opposite things.** Concentrated (a few accounts, especially enterprise/at-risk → relationship and churn risk, escalate) vs spread (many distinct accounts → systemic product gap, fix it). Compute the distinct account count and, when account facts are available, flag enterprise/at-risk from `health_label` (At-Risk) and `active_arr`. A theme concentrated in a high-ARR at-risk account with a near `days_to_renewal` is a churn-risk escalation, not just volume. This is intra-theme, so it's the producer's job, not the composer's.

### 5. Trend detection (vs the rolling baseline; steady-state suppressed; denominator-aware)
Compare each theme's current-window ticket count to its prior-window baseline. **Normalize against the `evaluated_items` denominator** so a change in coverage is not misread as a real support trend. Surface on **deviation**, by tier (ticket grain), using the tiers of the theme's product area:

- **≥ big_deal deviation → significant** ("big deal"): lead-worthy.
- **mention up to big_deal − 1 → worth a mention.**
- **< mention → filler**: emit low-weight; the composer's day budget decides whether it survives. The producer does **not** self-judge busy vs slow. (Run alone, show filler only in the "set aside" count.)

**Per-area tiers.** Each area's tiers come from its own history, so a low-volume area can still surface a trend. With `typical` = the area's typical weekly volume from `area_history`, scaled to the window (`typical × baseline_window_days / 7`), and the defaults in `area_tiers` and `thresholds`:

- `mention = min(thresholds.mention, max(mention_floor, ceil(mention_multiplier × typical)))`, by default `min(5, max(2, ceil(0.5 × typical)))`.
- `big_deal = min(thresholds.big_deal, max(big_deal_floor, ceil(big_deal_multiplier × typical)))`, by default `min(12, max(4, ceil(1.0 × typical)))`.

The tiers are low but never zero: an area with no tickets in its history still needs a deviation of 2 to be mentioned and 4 to be significant. High-volume areas hit the caps and use the absolute 12 / 5. **With no history for an area** (step 2b), use the floors (2 and 4) and say so in the coverage note. A theme that matches more than one area uses the tiers of the area it is emitted under (`product_area`). The `tier` field on each theme carries the result; the numbers used go in `coverage.note` (below), not in a new field.

A downward swing matters as much as an upward one: **a theme that FADED, especially one that dropped after a release shipped in that area, is surfaced as fix-confirmation** (*"timezone questions on recurring reminders dropped back to their usual one or two a week after last Tuesday's release"*), one low-key line, only when the drop is real against baseline.

Rules: **no blanket "new" signal**: a brand-new theme must still clear the volume bar. **Escalation is a trend, not a single**: a rising count of escalated/high-priority tickets in a theme is a **weight modifier**, never a trigger on its own. **Sentiment/state is a weight modifier too**: carry it forward to the composer even when it doesn't change surfacing (the composer correlates support slowness or volume with negative sentiment on the same area).

**Low-volume escape hatch (don't go dark on an early-stage area).** A notable *single* surfaces even below the volume bar when it's **high-priority / at-risk**, **or** it **corroborates a recent GA/release in the PM's area**. Still one ticket, framed as a single, never inflated into a "trend."

### 5b. Waiting-on-engineering handoff
**If `eng_wait_statuses` is empty or still the placeholder, skip this step**: emit `eng_waiting: []`, leave `eng_wait_queue` unchanged, and add "waiting-on-engineering status not set; stuck escalations not checked" to `coverage.note` (see Honesty).

A ticket in a status listed in `eng_wait_statuses` is a support escalation now in engineering's court. For the escalation candidates surfaced in steps 3–5, read the ticket's status, linked bug, and priority:

- **If a bug is linked:** hand the key to `engineering-progress-producer` (if installed) so it reports whether the bug is moving, how long it's sat, and its status. Run alone, report the linked bug with its link.
- **If no bug is linked, check before reporting it as having no linked bug.** An empty linked-bug field is not proof that no bug exists. If the ticket carries an escalation tag (for example a tag containing `jira` or `escalat`, such as `<your escalation tag>`), or the issue tracker is connected and a search for the ticket id finds an issue (Jira sites often have a custom field holding support ticket ids), treat it as below rather than claiming no bug exists:
  - Found in the tracker: treat it as linked (above) and set `linked_jira` to that key.
  - Escalation tag present but no issue found or no tracker connected: say *"escalated, no linked bug"*. Set `linked_jira: null`, `queued_for_jira: true` (so it lands in `eng_wait_queue` for `engineering-progress-producer` to find), stage it (🟡, `jira_defect` with detail "escalation tag present, no linked bug") and never raise it as 🔴.
  - No escalation tag and no issue found: set `linked_jira: null`, `queued_for_jira: true` (it lands in `support_trends.eng_wait_queue` for `engineering-progress-producer` to find or file), and flag *"waiting on eng, no linked bug."*
  - **What `linked_jira: null` means:** there is no linked bug. That is all it claims. Render it as *"no linked bug"*, never *"bug link unknown"* and never *"no bug filed"* (a bug may exist that nobody linked).
- **Compute the wait + SLA** per `sla_ref` (priority → target, severity → resolution SLA), and whether the same issue has recurred before (repeat = worse). A breach or near-breach on an at-risk enterprise account raises the weight; it is a **🔴** only when it has breached its SLA, has no linked bug, and carries no escalation tag. With no `sla_ref`, report days waiting and set `sla_status: null`; a ticket with no linked bug and no SLA status is staged (🟡, queue for bug) rather than 🔴, since 🔴 needs a known breach.

### 6. Emit structured output (delta-only; 💬 lens; no own section)

```
field_themes: [ { theme, product_area, feature?, current_count, baseline_count, deviation, evaluated_denominator,
                  tier: "significant|mention|filler", relevance: "strong|moderate|weak|null", staged: bool,   # staged=true for stage_only_areas or weak-tier
                  direction: "rising|falling", fix_confirmation: bool,
                  concentration: { shape: "concentrated|spread", distinct_accounts, enterprise_atrisk: bool, top_accounts: [ {name, active_arr, health_label, days_to_renewal} ] },
                  escalation_weight, sentiment_weight, representative_summary,   # theme-level; NO individual tickets; agent's read, NO customer quote
                  source: "<support tool>:tickets | paste:tickets", support_view_link } ]   # -> 💬 From the field & feedback
account_signals: [ { account, crm_account_id, open_count, escalated_count, active_arr, health_label, days_to_renewal, areas } ]   # null where no CRM/tool data; -> handed to the composer
eng_waiting: [ { ticket, link, account, priority, days_waiting, recurred: bool, linked_jira: "<KEY> | null", sla_status: "ok|at_risk|breached|null", queued_for_jira: bool } ]  # 🔴 only when breached + no linked bug + no escalation tag; linked_jira null = no linked bug; sla_status null = no sla_ref set; entries with queued_for_jira true and no linked_jira = support_trends.eng_wait_queue
cross_links: [ { theme, related: "feedback_theme|jira_item|adoption_signal", note } ]
staged_actions: [ { kind: "content_ticket|jira_defect|link_existing_idea", theme, area, target, detail } ]  # -> 🟡
coverage: { window, evaluated_items, note }        # so the composer never reads a thin pull as "quiet"; note also names each area's tiers and their basis
set_aside_count: <# deduped/multi-area collapses + steady-state themes + staged-not-surfaced not shown>
```

Each `eng_waiting` entry's `link` is the ticket's own URL: the source's ticket URL when it gives one, else built from `ticket_url_template` (or `org_constants.support_desk_domain`, see Config), else `null`, in which case the composer cites `support_ticket:<ticket>` in plain text.

`support_view_link` is a saved view or search URL in the support tool spanning the theme's tickets, when the tool supports one; otherwise the clearest single ticket URL; otherwise `null`.

**Provenance for the composer's de-dup** comes from fields already in the shape above, so there is no separate provenance field: each `eng_waiting` entry's `ticket` is its `support_ticket:<ticket>` key, plus `jira:<linked_jira>` when `linked_jira` is set; each `account_signals` entry's account key is `account:crm:<crm_account_id>` when `crm_account_id` is present, else `account:domain:<x>` from the requester's email domain, else the account name (the same bridge-key order as "Account facts" above; loose, never a merge). `field_themes` are aggregations of many tickets and carry no individual ticket ids (see `representative_summary`), so a theme dedups by subject, and `support_view_link` points at its contributing tickets. Emit real ids only; never fabricate.

`coverage.note` names the tiers used per area and their basis, for example *"Tiers: Plan & Track mention 5 / big deal 11 (typical 11/week over 8 weeks); Calendar Sync mention 3 / big deal 6 (typical 6/week, partial history of 5 weeks); Recurring tasks mention 2 / big deal 4 (floors; area added since the last refresh)."*

If nothing qualifies, emit one calm line ("no new support trends in your areas") **with the coverage note**; never pad, and never claim quiet off a thin pull.

**For a Product Lead (`role: PL`), run in EVIDENCE MODE, with no section of your own.** A PL doesn't triage support themes area by area. Emit the signal as **corroboration** for the composer (account / area / theme cross-link hooks), not standalone bullets. Only genuinely leader-level items surface **solo**: an **SLA-breached escalation on an at-risk account**, a **customer-facing or compliance defect with no linked bug on a recently launched feature**, or a **systemic theme concentrated in a major or at-risk account**. The composer owns whether and how a PL sees it.

## Standalone digest

When run alone (no composer present), render this in chat after the analysis. Show the structured block too only if the user asks for it or a composer is present.

```
💬 Support trends · <areas> · <window>
<One calm sentence if quiet, with the coverage note. On a heavy day, open with one sentence naming the biggest trend.>

 • <Theme, linked to its view or example ticket> is up <current> vs <baseline> tickets, <spread across N accounts | concentrated in Acme (At-Risk, renews in 40 days)>; <why it matters, one clause>.
 • <Faded theme> fell to ~<n> after <release>; that fix looks confirmed.

🔴 Needs you to act
 • <Ticket, linked> for <account> has waited <n> days on engineering with no linked bug<, past SLA>. (Only with no escalation tag and a known breach; an escalated ticket with no linked bug goes to 🟡.)

🟡 Needs your OK
 • <Staged action, one sentence>. [ Draft content ticket ] / [ Queue for bug ] / [ Link to idea ]

Coverage: <evaluated_items> tickets classified for <window>. Set aside <n> steady or duplicate items. Tiers: <area> mention <m> / big deal <b> (<from N weeks of history | capped | floors, no history>).< Waiting-on-engineering status not set; stuck escalations not checked.>
```

One complete sentence per item, the link on the phrase it names, significant before mention, no individual-ticket lists, no customer quotes. Omit 🔴 and 🟡 when empty.

## Actions: concrete and executable only

Only actions that map to an existing skill; no freeform advice. If a theme maps to none, it's context in the theme summary, never the 🟡 surface. If the skill for an action isn't installed, still stage it and say in one line that the skill exists at https://github.com/pendo-io/Product-Loop, offering to install it. To install: in Claude Code, or anywhere you can write files and reach GitHub, offer to do it yourself, and on the user's OK download that skill's folder from the repo into `~/.claude/skills/<skill-name>/`. Elsewhere (for example the Claude apps), tell the user to download the folder and upload it as a skill.

- **Docs / content gap** (how-to, troubleshooting, or confusion on a shipped feature) → stage a `content_ticket` action only if the user has a docs-ticket skill or tracker of their own; otherwise note the gap in the theme summary.
- **Defect trend** (recurring bug/error theme) **or a notable single defect on a shipped/GA'd feature** → **first check whether a bug already exists** (the linked-bug field from the waiting-on-engineering lookup). If one exists, cross-link it and report status (via `engineering-progress-producer` when installed). **If none, there is NO tracking**: queue it for `engineering-progress-producer` via `eng_wait_queue`. Only a live, observable defect becomes a bug; a defect that stopped reproducing is queued as an **engineering task to verify and harden it**, not as a bug. Any customer-facing or compliance defect with no tracking is worth surfacing.
- **Existing feedback** → **check whether a matching idea or feedback item already exists** (Pendo Listen via Pendo MCP, or another feedback tool, if connected) and, if so, stage an offer to **link the support theme to that existing idea** (hand to `feedback-idea-producer`'s linking when installed). Do **not** create a new one. With no feedback source connected, skip this action.

## Orchestrator notes

- **Class:** per-user **producer** (PM/PL, their areas). **Cadence:** on demand, or **daily** when scheduled or run by an orchestrator.
- **Digest buckets:** **💬 From the field & feedback** (primary; folded into the existing category, no section of its own); **🟡 Needs your OK** for staged actions and for `stage_only_areas`/weak-tier items; account facts feed the composer, which may raise **⚠️ Conflicts**. The composer owns that.
- **Producer ordering:** when `engineering-progress-producer` also runs, run this producer first so it can consume `support_trends.eng_wait_queue` in the same run.
- **Writes:** see Boundaries and authority.
- **Non-interactive on scheduled runs.**
- **Degrade gracefully:** if no ticket source is connected on an unattended run, emit nothing and name the gap; if no areas are configured, emit nothing and note it.

## Principles

- **Producer, not composer.** Emit signal tagged to the digest's *existing* 💬 category; add no section. Run alone, render a readable digest.
- **Trends, not tickets.** The theme is the unit; never surface individual tickets; steady-state is suppressed.
- **Classify to the user's areas** with names, aliases, tags, and feature names; stage what the matching can't yet trust.
- **Denominator-aware.** Every volume/trend claim is made against what was actually pulled and classified, and coverage is carried so a thin pull is never read as quiet.
- **Trust by area and account health**, not by relevance tier; use the tier only as a tiebreaker and to stage weak matches.
- **No customer quotes from support** (see Boundaries and authority).
- **Tie stuck escalations to the delivery signal** (step 5b).
- **Concrete actions only.** Every 🟡 maps to an existing skill.
- **The skill emits signal; the composer decides cross-signal conflicts.**
