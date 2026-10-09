# Call-signal retrieval modes: topic-first (default) → account-scoped → notes

The field producer isolates *how it pulls calls* into one swappable step. Everything downstream
(parse → mark the PM's own calls → dedup → map → cluster → weight → emit) is identical in every mode.
Retrieval **auto-detects** the best available tier at run time and degrades gracefully; the PM never
has to pick.

## Why the default is topic-first, not by-account

A PM cares about **their product area across every account**, not a hand-picked account list. The
account-scoped model exists as a *workaround* for call tools that answer one account or deal at a
time, so it needs a watch-list and misses any mention on an account the PM never flagged. Topic-first
search across all calls is the higher-value "I wouldn't have caught that" behavior.

## Tier 1: `topic_search` (default when a call tool supports cross-account transcript search)

- **Connector:** any call-recording or conversation-intelligence MCP or API that can search
  transcripts across all calls (not only per account).
- **Requires:** nothing account-specific, just the PM's `focus_areas` (turned into semantic
  queries). `field_qual.watch_accounts` is **optional** here and used only to *narrow* (a cohort
  filter), never to scope.
- **Retrieve:**
  - Cross-account transcript search, one semantic query per focus area (for example *"teams
    asking for timesheet reminders, or unsure who can approve a timesheet"*). Expect
    transcript **chunks with verbatim text, speaker tags (customer vs. your company), account name,
    call URL, and a relevance or distance score**. To narrow to a cohort, pass an account filter.
  - If your tool also offers a cross-source search, use it to corroborate a call theme. Record those hits in
    `cross_source_hits` (corroboration only; they don't count toward convergence).
- **Strength:** true cross-account discovery; **verbatim quotes and speaker affiliation come for
  free** (the chunks), which also makes PM-attended detection and real quotes possible, none of which
  the account tier can do.
- **Cost:** semantic search is broad. Start from the tool's own relevance or distance threshold and
  result count, tighten them when off-area hits crowd the results, and lead with convergence so the
  digest stays tight.

## Tier 2: `account_scoped` (fallback, per-account call tools)

Many call tools' public MCPs answer one account or deal at a time. Gong's official MCP is the common
example: `ask_account`, `ask_deal`, `generate_brief`. This tier must stay first-class, because it is
what many users have.

- **Requires:** `field_qual.watch_accounts` as `[{name, crm_id}]`. Name lookup is exact-match *and*
  can be ambiguous (one name can match several CRM records). An exact name that resolves to one
  account is answered silently and **no CRM id is returned**, so there is no read path to the id:
  store the name that resolved and leave `crm_id` null. When the tool returns a clarification
  response instead, pick the candidate with the most recent activity, confirm at setup, store the id
  it offers, and call by that id from then on.
- **Retrieve:** for each watch-account, in parallel, one natural-language question covering the
  curated keywords and aliases in the window, with sources included (in Gong's MCP,
  `includeSources: true`). Account not found: skip and note.
- **What it returns (do not over-promise):** **synthesized insights, not transcript**, grouped into
  themes, each with the calls it came from (id, title, URL, start time) when sources are included.
  Sources are attached per answer section, not per item, so several themes can cite the same call.
  No CRM id or domain comes back either, so the account provenance key falls back to
  `account:name:<name>` (see SKILL.md Step 9). So in this tier:
  - **No verbatim quotes.** Render a short *paraphrase attributed to the account and call*, never a
    fabricated quote.
  - **No speaker affiliation.** You cannot tell customer-said from rep-said; do not claim it.
  - **No moment deep-link and no participant list.** The link is **call-level** (`moment_link` is
    the call's bare URL, `link_scope: call`), and **PM-attended
    flagging is best-effort only** (infer from call title or known context; if participants aren't
    available, say "PM-attended: unknown" rather than guessing).
- **Strength:** reliable, no extra infrastructure, good for "watch my key deals."
- **Limit:** only sees watch-list accounts; less granular than Tier 1. A fallback, not the goal.

## Tier 3: `notes` (last resort, and the no-connector path)

No call tool connected: read transcripts or call notes the user pastes or uploads, or connected
meeting notes (Notion, Granola, or similar). A labeled transcript is parsed like a Tier 1 chunk
(verbatim, speaker affiliation); a summary is parsed like Tier 2 (paraphrase, affiliation unknown).
On an unattended run with nothing to read, emit nothing and name the gap; a composer may fall back to
its own meeting-notes source.

## Switching and detection

Resolve `field_qual.retrieval_mode` at run time: `auto` (default) → `topic_search` if a cross-account
transcript search is available, else `account_scoped` if a per-account call tool and `watch_accounts`
are present, else `notes`. A user can pin a mode, but `auto` is recommended so the skill upgrades
itself as soon as a better call source is connected. No pipeline change between tiers; only the
retrieve and parse steps differ, and parse normalizes every tier to the same hit shape
(quote-or-paraphrase, affiliation-or-unknown, account, call link).
