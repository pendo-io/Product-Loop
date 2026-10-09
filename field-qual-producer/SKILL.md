---
name: field-qual-producer
description: 'Produces a daily digest of the qualitative field signal from recent customer calls on a PM''s product areas: what customers and prospects say on calls the PM wasn''t on, and the recurring cross-account themes they''d miss. Searches all calls by topic in any connected call-recording tool, with verbatim quotes and speaker affiliation; falls back to a watch-list of accounts, then to pasted transcripts or notes. Counts calls the PM attended toward convergence but never surfaces them as new, and renders a short digest in chat plus a structured update a digest composer can fold in. Single owner of the cross-account customer-call signal for a PM''s areas. Not for single-account or deal research (use an account-research skill) or composing the digest (use attention-radar). Read-only on call, CRM and tracker tools; writes only its own config and, if opted in, one Slack post. Works on demand or daily. Triggers: "scan calls", "what are customers saying about X", "field signal", "call mentions", "run the field monitor".'
license: MIT. LICENSE has complete terms
compatibility: Standalone. No required connectors or skills; works from pasted call transcripts or notes. Optional connectors used if present, a call-recording tool with transcript search (for example Gong's official MCP), a feature tracker (Notion, Jira, or similar) to map hits to features, a CRM for deal name and stage, and Slack for an opt-in deeper digest. Optional companions, attention-radar and feedback-idea-producer.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

# Field (Qual) Producer: the customer-call signal

## When to use

Use for product-area, cross-account themes from recent customer calls the PM wasn't on. Triggers:
"scan calls", "what are customers saying about X", "field signal", "call mentions", "run the field
monitor". Runs on demand or daily. For quotes from a single account or deal, use an account-research
or sales skill instead.

## What it is

A **producer**: it owns one source (customer call transcripts) and produces a short, **delta-only**
update each run. It works two ways:

- **Run alone (default).** It renders its own readable digest in chat (see **Standalone digest**).
- **Inside a daily digest.** A digest composer such as `attention-radar`, or an orchestrator that runs
  producers daily, consumes its **structured output** (Step 9) and folds it into the digest's existing
  💬 From the field & feedback category. The producer then adds no section of its own. Both are
  available at https://github.com/pendo-io/Product-Loop. Neither is required.

Its job is to answer *"what are customers and prospects saying about my areas on calls I wasn't on,
and where is the same thing coming up across multiple accounts?"* The **cross-account convergence**
is the point: one theme surfacing on three accounts is the "I wouldn't have caught that" moment.
Whether that theme collides with or corroborates other signals (a support trend, a Jira break, a
feedback theme) is the composer's call. This producer emits its own signal and hands over the raw
facts.

"PM" below means whoever owns the product area; a **Product Lead** is someone who owns several areas
or has PMs reporting to them.

## Done

- **Result:** a short field-signal digest in chat (Standalone digest), or, under a composer, the
  Step 9 block.
- **Done when:** every surfaced item carries a call link or a plain `call:<id>`; every swept area
  is present or carries its "no call signal this window" line; `coverage_note` is present in
  account mode; `field_qual.reported` is updated, or the digest says dedup was not saved. A quiet
  run is the one calm line.
- **Next consumer:** `attention-radar` or a daily-digest conductor, which folds it into
  💬 From the field & feedback. Run alone, the person who asked.
- **Intent:** cross-account convergence over completeness. A quiet day beats a mislabeled hit.

## Boundaries and authority

Hard stops:

- Never present a paraphrase as a verbatim quote, because per-account tools return synthesized
  insight, not transcript; write it as a paraphrase attributed to the account and call.
- Never surface an off-area hit relabeled under the PM's area, because mislabeling is worse than a
  quiet day; count it in `set_aside`.
- Never claim "calls you weren't on" in account mode, because participants aren't returned, so it
  can't be guaranteed; say "across your watch-accounts".
- Never infer a deal from a call's content; leave `deal` null when no CRM or call tool supplies it.
- Never prompt on a scheduled run; use config, and if a required piece is missing, emit nothing and
  name the gap.
- Never fabricate ids, because provenance is the composer's de-dup key; pasted transcripts with no ids get `call:pasted-<n>` and are labeled.
- Never report a write as done without reading it back, because a failed write is silent: re-read
  `config.json` after each write and check the changed keys (a failed `reported` append means the
  digest says dedup was not saved), and check the Slack post's returned permalink.

**Authority.** Every write this skill can make:

- Its own `config.json` (`field_qual` namespace): setup values (keywords, watch-accounts,
  `slack_digest`, and the rest on the user's OK), `keywords` on `add` / `drop`, and the
  `reported` dedup append and 60-day prune every run that has a config file.
- Slack: one post of the deeper digest to `field_qual.slack_digest`, only when the PM opted in.
- Staged only: the "link to an existing Listen idea" 🟡, carried out after approval by
  `feedback-idea-producer` or by hand. This skill does not link it itself.

Read-only on the call tool, CRM, feature tracker, and meeting notes.

## Judgment vs mechanics

The model judges: the relevance gate (does a hit belong to the anchored area), theme clustering,
customer-said versus rep-said weighting, which area is a call's strongest, and phrasing. The call
tool owns retrieval, speaker tags, call ids, and links. These mechanics are still done by the model
from prose and are candidates for code: the surface and 14-day windows in the user's timezone,
tier-aware dedup keys, the 60-day prune, and convergence counting against
`convergence_min_accounts`.

## Honesty

- An empty result is valid: one calm line, never padded.
- Couldn't check is not nothing there: account mode carries its `coverage_note`; a missing account
  is skipped and noted; with no call source and nothing pasted, emit nothing and name the gap.
- Label what is partial: paraphrases are marked as paraphrases, `speaker_affiliation: unknown` and
  PM-attended "unknown" in account mode are stated rather than guessed.
- No time estimates.

**Before presenting**, check: no off-area hit left in; no paraphrase in quote marks; every
`moment_link` a bare URL or null; account-mode framing used in account mode; no PM-attended hit
surfaced as new. Fix what fails and check once more; if a check still fails, ship with the failure
named.

## Sources

- **Call transcripts (the core source).** Any connected call-recording or conversation-intelligence
  tool whose MCP or API can search transcripts (for example Gong's official MCP), or transcripts and
  call notes the user pastes or uploads. Retrieval auto-detects the best tier (see **Retrieval** and
  `references/call-retrieval-modes.md`).
- **Feature tracker (optional).** A Notion, Jira, Airtable, or similar features list, to map a hit to
  the PM's feature and narrow a shared area. Without one, hits map to the PM's `focus_areas` only.
- **Pendo Listen (optional).** If `feedback-idea-producer` or the Pendo MCP is available, a recurring
  product ask can be staged to link to an existing Listen idea.
- **CRM (optional).** Only to fill an account's deal name and stage (`accounts[].deal`). Read-only.
- **Slack (optional).** Only for the opt-in deeper digest.

## Config it reads (`config.json`, `field_qual` namespace)

If there is no config file, ask for what the run needs (focus areas, and the transcripts if no call
tool is connected) and offer to save it.

- `field_qual.retrieval_mode`: `auto` (default), `topic_search`, `account_scoped`, or `notes`. `auto`
  picks the best available tier at run time: a call tool with cross-account transcript search, else
  a call tool that only answers per account plus watch-accounts, else pasted transcripts or meeting
  notes (see **Retrieval**).
- `field_qual.watch_accounts`: `[{name, crm_id}]`. **Required only for `account_scoped`** (per-account
  call tools; about a 10-account cap). In `topic_search` it is **optional**, used only to *narrow* to
  a cohort, never to scope. `crm_id` is optional: per-account tools such as Gong's `ask_account`
  resolve an exact name silently and never return the CRM id, so store the name that resolved and
  leave `crm_id` null. When the tool returns a clarification response (the name was ambiguous), pick
  the candidate, store the id it offers, and call by that id from then on. Proposed, then confirmed
  and stored.
- `field_qual.keywords`: the curated, **alias-aware** watch-list
  `[{term, aliases[], source: "area"|"manual"}]`. Seeded from `focus_areas`, PM-curated, small. In
  `topic_search` these become **semantic queries** (natural-language, meaning-based); in
  `account_scoped` they're the exact terms and aliases the per-account question covers.
- `field_qual.reported`: dedup log, pruned after about 60 days. The key is **tier-aware**:
  `topic_search` dedups per (call_id + theme + moment ~10s); `account_scoped` has no moment, so it
  dedups per (call_id + theme).
- `field_qual.slack_digest`: optional `{channel}` for the **deeper Slack digest** (below); off by
  default, offered at setup.
- `field_qual.features_db`: optional pointer to the user's feature tracker (a database, board, or
  table), to map a hit to the PM's feature (relevance, cross-link, and narrowing when an area is
  shared). Read-only.
- `focus_areas`: seeds keyword suggestions and defines the PM's areas.
- `user`: the PM's identity (name, email), used to **flag calls the PM attended** (kept for theme
  convergence, marked not-new).
- `timezone`: the user's timezone. Day boundaries for the lookback windows (Step 1) use it.
  Defaults to the user's local time.
- `field_qual.convergence_min_accounts`: how many distinct accounts a theme needs to count as
  cross-account convergence (Step 7). Default `3`.
- `last_run`: when the previous run happened (under the conductor, `last_run.iso_timestamp` in the
  shared config). It bounds the surface window (Step 1); when it is absent, the run is a first run.

On a **scheduled run never prompt**: use config; if a required piece is missing, emit nothing and
name the gap (or let the composer name it).

Every run reads its state (`last_run`, `keywords`, `reported`) from config, never from the
conversation. Shape: `config.example.json` in this folder.

**Contract keys.** The output field names and provenance keys in Step 9 (for example `moment_link`,
`call:<id>`, `support_ticket:<id>`, `account:crm:<id>`) are stable, tool-neutral keys that composers
match on: `moment_link` holds the call link from whatever call tool you use (for example Gong), as
the bare URL only (never a label or prefix such as "call-level: "), or null when there is no link;
`link_scope` says what that link opens (`call` for the whole call, `moment` for the quote's
timestamp, null when `moment_link` is null); `support_ticket:<id>` a support-ticket id from whatever support tool surfaced it, and `account:crm:`
an account id from whatever CRM the call tool syncs with.

## The keyword model: area-anchored, alias-aware, PM-curated

Calls are **raw transcript**, so the keyword *is* the search query, and the watch-list is chosen
differently than for pre-tagged sources (support tickets, feedback). In `topic_search` the area
anchors become **semantic queries** (describe the pain in a sentence, meaning-based match); in
`account_scoped` they're literal terms and aliases in the per-account question.

- **Seed from `focus_areas` (product-area or product names), not internal feature titles.**
  On a call, people say "Timesheets", not the title of the epic that changed it or its codename.
  Area and product names are the words that come up.
- **PM curates to a small starting set** at setup (cap about 2 to 5; warn and offer to split when an
  area is too broad, for example all of "Plan & Track", or when the PM would drown in hits).
- **Alias-aware.** Store customer-facing names plus known codenames and transcription variants (for
  example a product name the recorder often transcribes as a similar-sounding word).
- **Manual add or drop** over time: `add <keyword>` / `drop <keyword>` in an interactive session
  updates `field_qual.keywords`.

## Retrieval: topic-first by default, account-scoped fallback

Retrieval is isolated so the rest of the pipeline is identical in every mode, and it
**auto-detects** the best available tier (never make the PM choose). Full detail in
`references/call-retrieval-modes.md`.

- **`topic_search` (default when a call tool supports cross-account transcript search).** Search
  across **all** calls for the PM's areas, no watch-list: one semantic query per focus area, returning
  verbatim transcript chunks with **speaker tags** (customer vs. your company), account, and call
  link. If your tool also offers a cross-source search, use it to corroborate a call theme.
  Optional account filter narrows to a cohort. This is the
  higher-value behavior: it catches a mention on **any** account and yields real quotes and
  affiliation.
- **`account_scoped` (fallback, for call tools that answer one account at a time).** For example
  Gong's official MCP `ask_account`. Query each `watch_account` **in parallel** with one question
  covering the keywords and aliases in the window, with sources included; **call by the stored
  name that resolved**, or by the stored `crm_id` when a clarification response supplied one (on an
  ambiguous match pick the most recent activity and confirm at setup). Account not found: skip and
  note. Sources come back per answer section, not per item, so several themes from one answer may
  all cite the same call; say so rather than implying each theme had its own call. These tools return **synthesized insight, not
  transcript**, so there is no verbatim quote, no speaker affiliation, and no moment deep-link
  (call-level link only).
- **`notes` (last resort, and the no-connector path).** No call tool connected: work from transcripts
  or call notes the user pastes or uploads, or from connected meeting notes (Notion, Granola, or
  similar). Treat a pasted transcript like a `topic_search` chunk when it has speaker labels and like
  `account_scoped` when it is a summary. On an unattended run with nothing to read, emit nothing and
  name the gap.

**Inspect the live tool response at first real run.** Don't hardcode field names; build the parse
against what the connected tool actually returns. Large results: parse with a **subagent** into
one-line-per-hit so raw transcript or JSON stays out of context.

## Setup (first run, interactive, then unattended)

Detect the retrieval tier first; setup depends on which one is live.

1. **Detect the tier.** A call tool with cross-account search: `topic_search` (no account setup
   needed; `focus_areas` are enough). A per-account call tool: `account_scoped` (needs
   watch-accounts). Neither: `notes`; ask the user to paste transcripts or notes, or name the gap.
2. **Keywords or semantic queries, covering the FULL area scope.** Propose candidates from
   `focus_areas`, the PM curates to a small alias-aware set (warn on broad areas), store. In
   `topic_search` these are phrased as semantic queries; in `account_scoped` as literal terms and
   aliases. **If the user is a Product Lead**, anchor on their *entire* led portfolio (every area
   they own) and sweep each; account for all of them in the output, including the quiet ones (render
   a swept-but-empty area explicitly, for example *"Recurring tasks: no call signal this window"*),
   never a convenient subset.
3. **Watch-accounts (only for `account_scoped`; optional cohort filter for `topic_search`).** Propose
   from the PM's committed or at-risk accounts where known, else ask for their top accounts;
   **confirm each name resolves** in the call tool (an exact name resolves silently with no id
   returned; if the tool returns a clarification response, disambiguate by most recent activity,
   confirm, and keep the id it offers), store `[{name, crm_id}]` with `crm_id` null when no id was
   returned. About a 10-account cap. Tell `account_scoped` users the watch-list is a
   limitation of per-account call tools, and that it lifts to all calls automatically once a call
   tool with cross-account search is connected.
4. **Optional deeper Slack digest.** Ask if they want the full grouped digest posted to a channel as
   well; store `field_qual.slack_digest = {channel}`. Default off.
5. **Optional schedule.** Offer a daily scheduled run if the host supports scheduled tasks. It works
   on demand either way.

## Daily workflow

### 1. Lookback windows: surface vs. convergence (two windows, on purpose)
The **surface window** (what counts as NEW to show) is 72h on a Monday, so Friday and the weekend
are covered, and 24h on other weekdays. Days start and end in the user's `timezone`, and the window
never reaches back past `config.last_run`. **Convergence uses a separate rolling window of about 14
days** over the hits kept so far, because the same theme across accounts rarely shows up inside one 24h
slice, especially in `account_scoped` where daily call volume per area is low. Only **new** hits
(surface window) are surfaced; the rolling set only raises a theme's `convergence_count` and can
promote it to the lead. Dedup (Step 5) keeps the same call and theme from re-surfacing day to day.
On an on-demand run with pasted transcripts, the surface window is the calls provided.
**First run (no `config.last_run`):** use the normal surface window for the day (72h on Monday, 24h
otherwise) and say so in one line ("first run: showing the last 24h; older hits count toward
convergence only"). Older hits in the rolling window still feed `convergence_count`.

### 2. Load config
`retrieval_mode`, `watch_accounts`, `keywords` and aliases, `reported`, `slack_digest`, `timezone`,
`convergence_min_accounts`.

### 3. Retrieve calls (tier-dependent, see Retrieval)
`topic_search` (default): cross-account transcript search, one semantic query per focus area (plus
a cross-source search to corroborate a theme if your tool offers one); optional account filter to narrow.
`account_scoped` (fallback): parallel per-account question per watch-account (by resolved name, or
`crm_id` when stored) for the
keyword set in the window, sources included (account not found: skip and note). `notes`: the pasted
or connected transcripts and notes. Large results: parse with a **subagent** into one-line-per-hit.

### 4. Parse hits: normalize every tier to the same shape
For each hit: `keyword/theme`, `account`, `call_id`/`title`, call link. Then, **tier-dependent**:
- `topic_search`: pull the **verbatim** snippet from the transcript chunk with its **speaker and
  affiliation** (customer vs. your company) and the call link (`moment_link`, with `link_scope`
  `moment` when the URL opens at the quote's timestamp, else `call`); a cross-source hit is
  recorded in the call theme's `cross_source_hits` with its source (Step 9). Cross-source hits
  corroborate only: they never count toward `convergence_count` or `accounts`, which come from
  calls alone.
- `account_scoped`: the tool returns **synthesized insight, not transcript**, so record a short
  **paraphrase attributed to the account and call** (never a fabricated quote), set
  `speaker_affiliation: unknown`, and use the **call-level** link (no moment deep-link):
  `moment_link` is the call's URL and `link_scope` is `call`.
- `notes`: verbatim only when the pasted text is a labeled transcript; otherwise paraphrase and
  `speaker_affiliation: unknown`. No link unless the user gave one (`moment_link` and `link_scope`
  null; when the user gave one, set `link_scope` by what it opens).

### 5. Mark the PM's own calls, then dedup
**Flag, don't drop, any call where the PM (`config.user`) is a participant.** They were in the room,
so it isn't a *new* discovery, but the theme still counts toward cross-account convergence: label the
hit `pm_attended` and never surface it as a missed or new moment. **Detection is tier-dependent:** in
`topic_search` check the speaker tags for the PM; in `account_scoped` participants aren't returned, so
PM-attended is **best-effort** (infer from call title or known context) and otherwise reported as
`unknown` rather than guessed. Then drop hits already in `reported` using the **tier-aware key**:
(`call_id` + theme + moment ~10s) in `topic_search`; (`call_id` + theme) in `account_scoped` (one
synthesized theme per call, so a call and theme seen before is not re-surfaced). Keep only new hits.

### 6. Relevance-gate, map to feature, flag internal-on-call
**Relevance gate first (required).** Semantic retrieval bleeds adjacent topics: per-account questions
are answered loosely, and transcript search is distance-ranked, so a query for one area routinely
returns comments about a *neighbouring* area (a Timesheets query pulling back complaints about
Calendar Sync double-booking, say). **Discard any hit that does not genuinely belong to the anchored area**;
count it in `set_aside`, and never surface an off-area hit relabeled under the PM's area (mislabeling
is worse than a quiet day). When in doubt, drop it. Then **map each surviving hit** to the PM's
feature or area (cross-link; narrow when an area is shared by multiple PMs), and flag calls with your
company's executives on them for manual review in `internal_on_call_flags`. That list is for the
user's review only: it is never customer signal, never counts toward a theme or convergence, and never
changes a hit's weight.

**One call can belong to more than one area. Keep every genuine vote, dedupe only the display.** A
single call may raise Timesheets *and* Workload View; the relevance gate removes only the *non*-genuine
matches (for example Calendar Sync remarks that a Timesheets query dragged in), never a real one. A
call that genuinely spans two areas contributes a hit, and a convergence vote, to **each** area it
truly belongs to; **votes are never dropped.** To avoid showing the reader the same call several
times, **dedupe the display, not the signal**: surface the call once under its **strongest** area and
note the others as a **cross-link** ("also raised: Workload View"), not a full repeat. The dedup ledger
keys on (call + theme), so a two-area call holds two entries and each theme tracks its own surfacing
independently.

### 7. Cluster and find convergence
Group into themes per keyword or area, and detect the **cross-account** pattern (same theme on at
least `field_qual.convergence_min_accounts` distinct accounts, default 3). Convergence is the lead-worthy signal. **Don't force a pain frame:** the convergent theme
can be **demand, validation, or praise**, not only friction. For an emerging area (a new product,
lots of discovery and demo calls) the strongest signal is often *pull* (prospects asking when they
can try it, a customer proposing a new use for it), and that is a valid lead. Surface what the calls
actually converge on; never manufacture a complaint, and never mislabel demand as a quiet day. **For a
Product Lead**, privilege the **cross-area, cross-account** convergence (a theme spanning several of
their areas and accounts is more leader-relevant than any single-area item) and lead with it.

### 8. Weight (not gate)
**Customer-said > rep-said**; convergence across accounts raises weight; a lone rep mention stays low.
Sentiment and urgency are **weight modifiers, not triggers**.

### 9. Emit structured output (💬; no own section inside a digest)

```
field_signals: [ { theme/keyword, feature, accounts:[{name,deal}], speaker_affiliation,
                   representative_quote (verbatim, short), moment_link, link_scope, convergence_count,
                   new_since_last_run, provenance,
                   cross_source_hits: [ { source, ref, account } ] } ]       # -> 💬 From the field & feedback; cross_source_hits corroborate only, never counted toward convergence_count or accounts
cross_links: [ { field_theme, related: "feedback_theme|support_theme|jira_item|adoption_signal", note } ]
internal_on_call_flags: [ { call_id, title, execs } ]                      # review-only list; never customer signal
staged_actions: [ { kind: "link_existing_idea", theme, feature, detail } ]   # -> 🟡
set_aside_count: <# not surfaced: steady / dedup'd / off-area (relevance-gated)>
coverage_note: <account_scoped only: what this run did NOT see: call-only, PM-attended unfilterable, N of M watch-accounts>
pm_attended_count: <# hits from calls the PM attended, counted toward convergence, flagged not-new>
```

**`provenance` (per field signal) is the composer's de-dup key**: `call:<call_id>` for each
contributing call (always use the call tool's **call id**; the call link in `moment_link` is often
missing, so treat it as an optional human link, never the key), plus an account key per account,
plus `support_ticket:<id>` when the signal came from a cross-source item that carries its own id. Account keys are **type-qualified**: `account:crm:<id>` when the call tool gives an
account id from Salesforce or another CRM, else `account:domain:<x>`, else `account:name:<account name>` when the call tool
returns neither (Gong's `ask_account` returns neither). `account:name:` is a loose correlator only:
the composer may relate items that share it, never merge on it, and it matches only other
`account:name:` keys. Other cross-source mentions that carry no artifact
id emit just the account key; let the composer soft-match those by subject. Pasted transcripts
with no ids get a stable local id (`call:pasted-<n>`) and are labeled as such. Always emit real ids;
never fabricate.

**`cross_source_hits`** lists the non-call items that corroborate a call theme: `source` is the
kind of item the cross-source search returned, using the item type label the tool gives it, `ref` its
id or URL as returned (null when it has none), `account` as returned. Empty when the tool has no
cross-source search. These hits never change `convergence_count`, `accounts`, or the lead.

**`accounts[].deal`** is the opportunity name and stage from a connected CRM (or from the call tool
when it returns them), for example `{ name: "Globex expansion", stage: "Proposal" }`. It is `null`
when unknown; never infer a deal from the call's content.

Per-tier provenance: in `topic_search`, `representative_quote` is **verbatim** and
`speaker_affiliation` is real; in `account_scoped` it is a short **paraphrase attributed to the
account and call**, `speaker_affiliation` is `unknown`, and `moment_link` is the call's URL with `link_scope: call`. In
every tier `moment_link` is a bare URL or null, never a labeled string. Never
present a paraphrase as a verbatim quote.

**Account-mode coverage honesty (required).** `account_scoped` sees fewer things than it sounds like,
so it must say so and must not over-claim:
- Append a one-line `coverage_note`, for example *"account mode: call-only, other sources not
  seen · PM-attended can't be filtered · scanned N of your M watch-accounts."* (M is the number of
  `watch_accounts` in config; N is how many the call tool found.)
- **Drop the "calls you weren't on" framing in account mode**: participants aren't returned, so it
  can't guarantee it. Use *"across your watch-accounts"* instead. (Only `topic_search`, which has
  speaker tags, may claim "calls you weren't on.")
- The gaps hurt most when one PM's area touches most of the customer base: account mode
  can't filter their own calls, misses signal from other sources, and can't watch-list their
  whole area. Name those gaps rather than implying full coverage.

If nothing qualifies, emit one calm line ("no new field signal in your areas"); never pad.

When a composer is present, hand it this block. When run alone, render the **Standalone digest**
below, and include this block only if the user asks for it.

### 10. Optional deeper Slack digest
If `field_qual.slack_digest` is set, also post the **full grouped-by-keyword digest** (every mention,
account, speaker, deep-link) to that channel. The daily digest stays the concise convergent view; the
Slack channel is the full archive for PMs who want it. Nothing else is ever posted, and this post
happens only on the PM's opt-in.

### 11. Update dedup
Append newly surfaced hits to `field_qual.reported` (prune 60d) in `config.json`, using the same
tier-aware key (Step 5), so the next run doesn't re-surface a call and theme. If there is no config
file (a one-off on-demand run), skip this and say dedup was not saved.

## Standalone digest (when run alone)

Render markdown in chat, built only from Step 9's output:

```
💬 From the field & feedback: field signal · <areas> · <surface window>
<one-sentence lead, only if a convergent cross-account theme exists>

 • <theme>: one complete sentence on what customers said and why it matters, with the
   [account or call](call link) linked on the phrase, "verbatim quote" (customer) or
   a paraphrase marked as such, and the convergence count.
 • <swept-but-empty area>: no call signal this window.   (Product Leads)

🟡 Needs your OK
 • <staged action>, for example link this recurring ask to the existing idea [idea name](idea link),
   with a one-clause reason (for example "raised on 3 accounts this week").

Set aside N (steady, already reported, or off-area). <coverage_note, if any>. <pm_attended_count> hits were from calls you attended (in account mode: "PM-attended: unknown").
```

Rules: one complete sentence per item (a convergent theme may run to two); link the phrase itself to
the specific call or artifact, never a section-level URL, and cite by id in plain text when no
complete link exists; state what was said, not an inferred why; don't tell the reader what to do.
Omit 🟡 when empty. A quiet day is the one calm line.

## Actions: concrete and executable only
Field signal is mostly **FYI and awareness** (calm register). The only staged 🟡s map to existing
skills: a recurring product ask on calls → offer to **log or link it to an existing Pendo Listen
idea** (hand to `feedback-idea-producer` if installed; otherwise show the ask and the idea to link by
hand). No freeform advice, no account Slack pings.

## Notes for composers and orchestrators
- **Class:** per-user **producer** (PM or Product Lead, their focus areas; watch-accounts only in the
  account fallback). **Cadence:** daily when scheduled, or on demand.
- **Digest buckets:** **💬 From the field & feedback** (primary; folded into the existing category, no
  section of its own); **🟡** for the staged actions above; convergent themes and account facts feed
  the composer, which owns any **⚠️ Conflicts** determination.
- **Boundary with the composer:** this is the call-qual source feeding 💬. When it runs, the composer
  narrates it instead of scanning calls itself; the conductor passes the resolved call-notes source;
  this producer picks its own retrieval mode.
- **Writes:** see Boundaries and authority. Nothing is written to the call tool or the feature
  tracker.
- **Non-interactive on scheduled runs.**
- **Degrade gracefully:** auto-pick the tier: cross-account search → `topic_search`; per-account tool
  → `account_scoped`; else `notes`, or emit nothing and name the gap. No focus areas to anchor on →
  ask (interactive) or emit nothing and note it (unattended); no watch-accounts in `account_scoped` →
  setup prompt, quiet on an unattended run; account not found → skip and note; ambiguous → use the
  id from the clarification response (confirm at setup) or skip and note.

## Principles
- **Producer, not composer:** emit signal tagged to the existing 💬 category; add no section inside a
  digest, and render a short readable digest when run alone.
- **Discovery, not a recap:** mark the PM's own calls as attended (they add to convergence, not
  surfaced as new); lead with the convergent cross-account theme.
- **Customer voice over rep chatter:** attribute affiliation; weight customer-said above rep-said.
- **Area-anchored, alias-aware, PM-curated keywords:** product and area names customers actually say,
  not internal feature titles.
- **Topic-first, account-fallback:** default to semantic search across all calls so a PM sees their
  area everywhere, not just a watch-list; fall back to per-account questions, then to pasted notes.
  Same downstream pipeline; the parse step normalizes every tier.
- **Config-driven, state in config.json:** watch-accounts, keywords, and dedup all live in config.
- **Concrete actions only:** every 🟡 maps to an existing skill (link-to-idea); no freeform advice.
