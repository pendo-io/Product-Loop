---
name: competitor-release-producer
description: 'Produces a digest of competitor releases relevant to a PM. Reads the competitor releases you point it at (pasted release notes, changelog or release-notes pages, RSS feeds, or your own competitor-release tracker in Notion, Airtable, or similar), keeps the ones your tracker marks relevant (when it triages them) that fall in your product areas or that name you as an owner, spots what is NEW since the last run, and renders a short digest in chat. It also emits a delta-only structured update that a digest composer such as digest-composer folds into the daily digest''s 🧭 Competitive watch category. Single owner of the 🧭 Competitive watch lens and the per-release relevance question. Not for composing the daily digest; use digest-composer. Read-only on every source; writes only its own config. Use for a PM''s daily or on-demand competitive read. Triggers: "what did competitors ship", "competitor releases", "competitive watch", "what''s Competitor A doing", "competitor signal in my area".'
license: MIT. LICENSE has complete terms
compatibility: Standalone. Works on pasted release notes with no connectors. Optional sources, used if connected, are a web fetch tool (changelogs, release-notes pages, RSS), and a Notion, Airtable, or similar connector for your own competitor-release tracker or feature tracker. Optional companion, digest-composer, composes its output into a daily digest.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

## When to use

Use for a PM's competitive read, daily or on demand: the relevant competitor releases in their areas
they might otherwise miss. Triggers: "what did competitors ship", "competitor releases", "competitive
watch", "what's Competitor A doing", "competitor signal in my area". Read-only on every
source and writes only its own config; runs daily if you schedule it, and works just as well when asked.

# Competitor Release Producer

A **producer**: it owns one signal (competitor releases) and each run hands over a short, structured,
**delta-only** update. It works in two ways:

- **Standalone (default).** Run on its own, it renders its own readable digest in chat (see "Standalone
  digest" below). Nothing else needs to be installed.
- **Feeding a daily digest.** When a digest composer such as `digest-composer`, or an orchestrator that
  runs producers daily, is present, it hands over the structured output instead, and the composer folds
  it into the daily digest's **🧭 Competitive watch** category. This producer adds no section of its own
  in that mode. `digest-composer` is in https://github.com/pendo-io/Product-Loop. A setup from before
  its 1.1.0 rename may have the full composer under its old name; that copy counts as installed too (a
  deprecated stub, whose description starts with "Deprecated", does not). If no composer is installed,
  say so once and offer to install `digest-composer`, never block.

Its job: answer *"which competitor releases landed in my areas that I might have missed: who shipped
what, and does it map to a feature I own?"* It reads releases at the source you configure, so the PM
gets the release itself.

## Done

- **Result:** a short 🧭 Competitive watch digest in chat (step 7), or, under a composer, the
  structured object in step 6.
- **Done when:** every surfaced release has a linked title (or competitor and title in plain
  text), a `via` reason, and a full-sentence `one_line`; `set_aside_count` carries its reasons; or
  the run is the calm line, or a named gap. When config is saved, `competitor_release.last_run` and `seen` are written
  back; when it isn't, the digest says so.
- **Next consumer:** `digest-composer` or the `daily-digest` conductor, which folds it into
  🧭 Competitive watch. Run alone, the person who asked.
- **Intent:** relevant and new, not everything. When steps run out, set aside rather than surface
  an out-of-area release.

## Boundaries and authority

Hard stops:

- Never invent a release; with no source reachable, ask for pasted notes (attended) or emit nothing
  and name the gap (unattended).
- Never create or alter tracker properties; read the schema first and map columns via
  `competitor_radar.fields`.
- Never invent a triage rubric for sources with no triage; ask the user "Is this relevant to you?"
  and save the answer.
- Never filter Product Area on an exact string, because it is free text that drifts from your area
  names; fuzzy-match it.
- Never hard-merge a competitor release with another producer's item, because they share no
  artifact id; relate by subject.
- Never invent a provenance id; omit `provenance` when there is no title and no URL to build one.
- Never prompt on a scheduled run; stage the relevance questions and never wait for an answer.
- Never report config as saved without reading it back, because a failed write would resurface
  seen releases next run; re-read `config.json` after each write and check the changed keys, and
  confirm an installed composer's `SKILL.md` is on disk (`digest-composer`, or the full composer
  under its pre-1.1.0 name).

**Authority.** Every write this skill can make:

- Its own config (`config.json` next to the skill, or the conductor's shared `competitor_radar`
  block): `competitor_release.last_run`, plus `seen` and `triage_answers`, after each run, only when
  config was saved.
- Setup answers (focus areas, competitors, sources) saved to `config.json` only when the user
  agrees.
- A companion install: `digest-composer`, only when the user accepts the offer.

Read-only on every source (tracker, changelog, RSS, feature tracker).

## Judgment vs mechanics

The model judges: the fuzzy area match, a related feature's tie to an owned feature, patterns
across competitors, ordering, and phrasing. Connectors and web fetch gather the releases. These
mechanics are still done by the model from prose and are candidates for code: release-id
normalization, `seen`-list dedup, the delta cutoff (created versus published, and the first-run
window), and set-aside counting.

## Honesty

- An empty result is valid: one calm line carrying direction, never padded.
- Couldn't check is not nothing there: an unreachable source is skipped and named; an unresolved
  `features_db` keeps the free-text related feature as a soft hint.
- Label what is inferred: a Product Area inferred from release text is labeled inferred, and an
  unclassified release stays "unclassified" until the user answers.
- No time estimates.

**Before presenting**, check: every title linked or cited in plain text; no out-of-area release
surfaced; unclassified items labeled; set-aside reasons present. Fix what fails and check once
more; if a check still fails, ship with the failure named.

## Sources (bring your own)

`competitor_radar.sources` is a list of typed entries. Use whichever are configured, in this order of
trust, and fall back to asking for pasted input when none resolve:

| type | what it is | needs | notes |
|---|---|---|---|
| `tracker` | your own competitor-release database (Notion, Airtable, a spreadsheet) with one row per release | that tool's connector | read-only; read its schema first and map columns via `competitor_radar.fields`; never create or alter properties |
| `changelog` | a competitor's changelog or release-notes page URL, per entry in `competitors` | a web fetch tool | each dated entry on the page is one release |
| `rss` | a release or blog RSS/Atom feed URL | a web fetch tool | each item is one release |
| `pasted` | release notes the user pastes into chat | nothing | always available; the no-connector path |

If a configured source can't be reached (no connector, no web tool, a fetch fails), skip it, name it in
the run notes, and continue with the rest. With **no** source reachable, ask the user to paste release
notes (attended runs), or emit nothing and name the gap (unattended runs). Never invent a release.

## Config it reads (never hardcode identity)

All identity comes from config; nothing is hardcoded. See `config.example.json`.

**Where the config lives.** Run alone, the producer keeps its own `config.json` next to the skill.
When the `daily-digest` conductor runs it, the `competitor_radar` block (competitors, sources, fields,
`releases_db`, `competitors_watch`, `delta_anchor`, the seen list, and saved triage answers) lives in the conductor's shared config instead, so the user
sets it once; read it from there and write `seen` and `triage_answers` back there. The delta anchor
always lives in this producer's own section, `competitor_release.last_run`, in whichever file is in
use; never read or write the conductor's top-level `last_run`, which records the conductor's own run. If
both exist, the shared config wins and the producer's own file is left untouched.

- `competitor_radar.competitors` — the competitors you track, each `{ name, sources: [ { type, url } ] }`.
  Required for `changelog` and `rss` sources; with a `tracker`, the tracker's **Competitor** column is the
  list. Example names: Competitor A, Competitor B, Competitor C.
- `competitor_radar.sources` — the typed source list above (for a `tracker`: `{ type: "tracker", tool,
  ref }`, where `ref` is the database or table link).
- `competitor_radar.releases_db` — shorthand for a single `tracker` source (kept for compatibility with
  orchestrators that set it). There is no default: if neither resolves and no other source is configured,
  fall back to pasted input or name the gap. Never guess a database.
- `competitor_radar.fields` — maps your tracker's column names to the fields this skill reads (below).
  Unmapped fields are treated as absent.
- `competitor_radar.features_db` — optional: your **feature tracker** (Notion, Jira, Airtable, or
  similar), used to resolve a release's **related feature** to a real feature record so the release can
  be cross-linked to a feature the user owns. If unset, fall back to a soft name match and don't guess a
  record.
- `competitor_radar.window_days` — how far back the display looks (default **7**). When that window holds
  zero releases, it widens itself to 14. Delta itself is anchored on `competitor_release.last_run`, not this window.
- `competitor_radar.triage_filter` — which **Triage** values to surface. Default
  **`["relevant"]`**, matched case-insensitively against the tracker's values (so a column that reads
  "Relevant" matches): honor the upstream triage done by whoever curates the tracker; "Not relevant" is
  theirs to exclude, so it stays out unless the user opts it in. See "Triage" for sources that
  carry none.
- `competitor_radar.match_by` — how a release is judged relevant to this user. Default
  **`["area", "owner"]`**: an area match (the release's **product area** maps to one of the user's
  areas) **or** the user appearing in the release's **owners** (whoever the curator named, PM or not).
- `competitor_radar.competitors_watch` — optional subset of the tracked competitors to narrow to.
  Default: all.
- `competitor_radar.delta_anchor` — `created` (default) or `published`. See the delta rule below.
- `competitor_radar.first_run_days` — how far back the first run looks (default **30**). See the delta rule.
- `competitor_radar.seen` — the release ids (see "Release id" below) already seen on earlier runs, for
  `changelog`, `rss`, and `pasted` sources. Used to drop already-seen entries and to date **Added on**.
  Written by this producer after each run, only when config is saved. Starts empty.
- `competitor_radar.triage_answers` — the user's saved answers to the relevance question, keyed by
  the release's `competitor_release:<id>` with the same release id (`"relevant"` or `"not_relevant"`),
  so each answer applies to one release only. Written only from the user's answers; reused on every
  later run. Starts empty.
- `focus_areas` — the user's product area(s); the coarse area anchor. The user's features in
  `features_db` (and the product areas they belong to) are the authority when resolvable; `focus_areas`
  is the fallback.
- `role` — `PM` (default) or `PL` (a product lead who owns a portfolio of areas); passed through so the
  composer can pitch the lens at the right altitude (below). This producer emits the same signal either way.
- the **user identity** (`user.name`, `user.email`, and the tracker's person id if it has one) — for the
  owner match.
- `competitor_release.last_run` — when this producer last ran (an ISO timestamp, kept in this
  producer's own section of whichever config file is in use).

On a **scheduled run never prompt**: use config; if a required piece is missing, emit nothing and name
the gap.

## Data model (what each release carries)

Whatever the source, normalize each release to these fields. For a `tracker`, they come from the mapped
columns; for `changelog`, `rss`, and `pasted`, fill what the text supports and leave the rest empty.
Never fill a field by guessing.

- **Title** — the release headline.
- **Triage** (`relevant` | `not_relevant`; a tracker may spell them "Relevant" and "Not relevant",
  matched case-insensitively) — the curator's upstream triage, when the tracker has it.
  This is the primary noise filter.
- **Competitor** — who shipped it.
- **Product Area** (free text) — the area of *your* product the release bears on. Usually free text, so it
  drifts from your area names; match it **fuzzily** against the user's areas, never as an exact lookup.
  For non-tracker sources, infer it from the release text against the user's areas and label it inferred.
- **Related feature** (free text) — the feature of yours the release maps to, if the curator identified
  one. A **cross-link hint**, not a relation; resolve it to a real feature record only via `features_db`,
  else keep it as a soft, labeled hint.
- **Summary** — a short summary of the release.
- **Published** (date) — when the **competitor** shipped it (recency).
- **Owners** (free text) — the people the curator named for this release; match loosely against
  the user identity. Absent for non-tracker sources.
- **Tags** — provenance tags on the source item, if any.
- **URL** — the source link for the release. This is the deep link the digest renders; use the entry's
  own anchor link (`<page URL>#<anchor>`) when the page has one.
- **Added on** — when the release became known to us: the tracker row's created time, or, for
  `changelog`, `rss`, and `pasted`, the first run that saw it (recorded in the seen list in `config.json`).

**Release id (unique per release).** Several changelog entries often share one page URL, so the URL
alone is not an id. Build each release's id this way, first that applies:
1. A `tracker` row: the row's id.
2. An entry with its own link: an RSS item link or guid, or the page URL plus the entry's anchor
   (`<page URL>#<anchor>`), when that link is unique among the entries read this run.
3. Otherwise: `<page URL>|<normalized title>|<published date>`, where the normalized title is
   lowercased with punctuation removed and whitespace collapsed, and the date is `YYYY-MM-DD` (omit
   the date part when there is none). For pasted text with no URL, use `pasted|<normalized
   title>|<published date>`.
Use this one id everywhere: in `provenance` (`competitor_release:<id>`), in the `seen` list, and as the
key for `triage_answers`, so a relevance answer for one entry never applies to its neighbours on the same
page.

**Relational fields are usually free text.** Product Area, related feature, and owners are
typically text, so scoping is **in-memory fuzzy matching after the pull**, not a structured query filter.
Only **Triage**, **Competitor**, and the dates are reliably filterable at query time.

## Triage

- **Tracker with a Triage column:** honor it through `triage_filter` (default `["relevant"]`).
- **Source with no triage** (`changelog`, `rss`, `pasted`, or a tracker without the column): this
  skill does **not** invent a triage rubric. It asks the user instead:
  1. Look up each in-scope release in `triage_answers`. A saved `"relevant"` answer makes it a relevant
     release; a saved `"not_relevant"` answer sets it aside like a tracker's Not relevant row.
  2. An in-scope release with no saved answer is **unclassified**: apply the scope filter below as
     the only noise filter, pass it through with `triage: "unclassified"`, and stage a yes/no
     question, *"Is this relevant to you?"*, as a `triage_question` staged action.
  3. When the user answers, save the answer in `triage_answers` and reuse it from then on. The release
     keeps its `unclassified` label until answered.
  In the standalone digest, mark each unclassified item "unclassified" and batch the questions into one
  🟡 line. On a scheduled run, stage the questions and never wait for an answer.

## Delta rule — new *to us*, dated by the competitor

Two different dates, two different jobs:

- **Added on** (default `delta_anchor: created`) is the **delta anchor**: a release is **NEW** when it became
  known to us after `competitor_release.last_run`. A release the competitor shipped a month ago but that only reached the
  tracker (or the feed) today is **new to the PM**, and should surface. This is the right default: the
  PM's "new" is "new to our awareness," not the competitor's ship date.
- **Published** is the **recency label** for display and ordering ("shipped 3 days ago" vs "shipped in
  Q1"). Carry it through; don't use it as the delta gate unless `delta_anchor: published` is set (a user
  who wants strictly recently-*shipped* items).

On the **first run** (no `competitor_release.last_run`), look back `first_run_days` (default **30**) by Published, or by
Added on when there is no Published date; everything in that window is new. For pasted input on a run
with no `competitor_release.last_run`, everything pasted is new, capped to the same 30-day window by Published when dates
are present.

Older releases are context, not surfaced, unless they form part of an escalating pattern (for example a
third competitor shipping the same capability this month: surface the *pattern*, not the old releases).

## Scope — the user's areas + owner

Scope is `match_by` (default area OR owner):

1. **Area match.** Read the user's features in `features_db` and the product areas they belong to; fall
   back to `focus_areas` if features can't be resolved. Fuzzy-match each release's free-text **Product
   Area** to that set, tolerating casing, punctuation, and known drift (for example "Plan and Track" ≈ "Plan & Track",
   "resource planning" ≈ "Workload View"). Never filter off an exact string.
2. **Owner match.** If the user appears in the release's **owners** (by name or email, loose
   match), it's in scope regardless of area: the curator named them deliberately.

A release that matches neither is out of scope for this user; count it in `set_aside_count`, don't
surface it. Respect `competitors_watch` if set (drop releases from competitors not on the watch list).

## Setup

No heavy onboarding. On the first attended run, if config is missing, ask once, in one message, for:
the user's product areas, the competitors to watch, and where releases come from (a tracker link, a list
of changelog or RSS URLs, or "I'll paste them"). Offer to save the answers to `config.json`. If the user
just pastes release notes and asks, proceed with those and the areas they name. When an orchestrator
supplies `focus_areas`, `role`, and the user identity, reuse them and don't ask again. On an unattended
run with a required piece missing, emit nothing and name the gap rather than erroring.

## Workflow

### 1. Resolve scope
Read the user's features and their product areas (fall back to `focus_areas`). Build the run's fuzzy
area matcher, and the owner matcher from the user identity.

### 2. Delta pull (structured filters only)
For a `tracker`: query it filtering on what it reliably supports: **Triage IN
`triage_filter`** (when the column exists), optionally **Competitor IN `competitors_watch`**, and a
date cutoff on the `delta_anchor` (created since `competitor_release.last_run`, or Published within `window_days` if
`delta_anchor: published`). Sort newest-first. **Product Area / related feature / owners are free
text: do not attempt to filter them in the query;** pull the candidate rows and filter them in memory
next. For `changelog` / `rss`: read each competitor's entries, build each entry's release id, drop the ids already in `seen`,
and apply the same date cutoff. For `pasted`: parse each release out of the text. On the first run, cap
the look-back to `first_run_days` (default 30) rather than the whole history.

### 3. Scope filter (in memory)
Keep only releases that pass the `match_by` test (area fuzzy-match OR owner). Record why each kept
release is in scope (`via: area | owner`). Everything dropped here (out-of-area, not relevant,
already-seen) goes to `set_aside_count` with the reason, never silently.

### 4. Resolve the tie to your roadmap (cross-link hint)
For each kept release with a **related feature** (or, for unclassified sources, a clear feature match in
the text), try to resolve it to a real feature record in `features_db` (exact-ish title match, scoped to
the user's areas). If it resolves to a feature the **user owns**, or one another producer touched this
run (an engineering-progress delivery item, a feedback-idea theme on the same feature), emit a **soft**
`cross_link` the composer can relate. A competitor shipping the same capability the PM is building or
shipping is the highest-value competitive signal. Keep it a **relate, never a merge**: there's no shared
artifact id between a competitor release and your own work, so the composer correlates by subject and
labels it; it does not collapse them.

### 5. Weight & order (do not self-censor by day)
Order within the lens by: an **owned-feature match** first (a competitor shipping against something the
PM owns), then **owner** releases, then recency (Published), then the rest. Carry a weight; the
composer's day budget decides how many survive on a heavy day. Don't judge busy vs slow yourself.

### 6. Emit the structured producer output
Produce a compact object a composer can narrate:

```
category: competitive_watch            # -> 🧭 Competitive watch (conditional; omitted when empty)
headline: { competitor, title, one_line (full sentence), related_feature, product_area, published, url, provenance } | null
releases: [ up to ~5, each {
    title, competitor, product_area,
    triage: "relevant" | "unclassified",        # "unclassified" until the user answers the relevance question (sources with no triage)
    related_feature | null,
    owned_feature: { id, title } | null,        # resolved via features_db when the release maps to a feature the user owns
    summary, published, added_on,               # published = competitor ship date; added_on = when it became known to us (delta)
    match: { via: ["area"|"owner"], area_match, owner_match },
    one_line,                                    # complete sentence: who shipped what + the tie to your roadmap
    url, provenance,
    cross_link_hint: { feature, related: "owned_feature|jira_item|feedback_theme", note } | null
} ]
patterns: [ { desc, competitors: [], area, note } ]   # e.g. "3rd competitor to ship X in your area this month"
cross_links: [ { release, related: "jira_item|feedback_theme|owned_feature", note } ]
staged_actions: [ { kind: "triage_question", release, question: "Is this relevant to you?", url, provenance } ]   # -> 🟡 one yes/no per unclassified release
set_aside_count: <# releases not surfaced: not relevant + out-of-area + already-seen>
```

**`provenance` is the composer's key.** Each release carries `competitor_release:<id>`, where `<id>` is
the release id defined under "Data model" (the tracker row's id, or, for `changelog`, `rss`, and
`pasted` sources, the entry's own link or the page URL plus normalized title and date). If a release
has no title and no URL to build an id from, omit `provenance` rather than invent one. A competitor release shares **no** artifact id with any
other producer's item, so provenance here is used to de-dup *this* producer across runs and to anchor the
composer's cross-links by subject, never to hard-merge with a Jira, Listen, or support item.

Keep it **discovery, not recap**: only what's new and relevant since last run. If nothing new and relevant
landed, emit a single calm line carrying direction (for example *"nothing new from competitors in your areas
today; 4 releases set aside as not relevant or out of area"*) and let the composer omit the conditional lens. Never pad, never
invent a release, never surface an out-of-area release to fill space.

**Rendering contract for the composer:** write `one_line` as a **complete sentence** that leads with the
competitor and what they shipped, then the tie to your roadmap, for example *"Competitor A
added leave-aware capacity to its resource planner, which maps to your Workload View area."* Link the **title phrase** to
the release's **URL**; every release must carry a complete deep link. If a release has no URL, cite it by
competitor + title in plain text rather than a bare database or homepage link. When a release maps to an
**owned feature** or corroborates another lens, say so in the sentence and carry the `cross_link` so the
composer can point the two at each other (competitive pressure ↔ the feature in 🚢 Shipped & in-flight).

### 7. Standalone digest (when no composer is present)
Render the structured output as a short markdown digest in chat:

```
🧭 Competitive watch · <areas> · <date>
<headline one_line, if there is one>

 • [<title>](<url>): <one_line>  (shipped <published>; <via area | owner>; unclassified if so)
 • ...
 Pattern: <patterns[].desc>, if any.

🟡 Is this relevant to you? <unclassified release titles, each linked>: yes or no for each.

Set aside <set_aside_count> (not relevant, out of area, or already seen).
```

One complete sentence per release, the title linked to its source, owned-feature matches first. On a
quiet run, show only the calm line. Include the structured block too when the user asks for it or a
composer will consume it. Afterward, update `competitor_release.last_run` and add each release id to `seen` in the config if the user
saved config; otherwise mention that without saved config every pasted release counts as new next time.

## PM vs PL — same signal, different altitude
- **PM:** the full 🧭 Competitive watch lens as above: the relevant releases in their areas, owned-feature
  matches leading.
- **PL (evidence mode):** a PL doesn't read a per-release list. Surface only the **cross-cutting**
  competitive picture: a capability **multiple competitors** are now shipping across the portfolio, or a
  single release with **strategic** weight on an area they own, synthesized, not enumerated. A lone
  release surfaces solo only when it's leader-critical (a competitor directly targeting a committed bet).
  When a composer is present it owns PL composition; this producer hands over the signal and the pattern
  hooks. Standalone, apply the same rule to the digest.

## Running with a composer or on a schedule
- **Class:** per-user **producer** (PM/PL, their focus areas). **Cadence:** daily when scheduled; on demand
  otherwise. The 🧭 Competitive watch category is conditional, so a quiet day shows nothing there.
- **Digest category:** **🧭 Competitive watch** (a **conditional** category, shown only when relevant
  releases land, like ⚠️ Conflicts; no "nothing" line on quiet days; may drive the headline on a heavy
  competitor day). Folded in by the composer; this producer adds no section of its own in that mode.
- **Writes:** see Boundaries and authority. It writes nothing to any source. Its only staged items
  are the relevance questions.
- **Non-interactive on scheduled runs.**
- **Degrade gracefully:** if a source can't be reached, skip it and name the gap; if `features_db` can't be
  resolved, skip the owned-feature resolution and keep the release with its free-text related feature.

## Connectors (all optional)
- **Web fetch** — read `changelog` and `rss` sources.
- **Notion, Airtable, or similar** — read your competitor-release tracker (delta pull, filtered on
  Triage / Competitor / date), and optionally your feature tracker to resolve a related feature to
  an owned feature. Read-only.
- **None** — paste release notes into chat.

## Principles
- **Producer first** — emit signal tagged to the 🧭 Competitive watch category; standalone, render it as
  a short digest.
- **The source is the source of truth** — read releases at the source, honor the tracker's triage where
  it exists, and don't invent a triage where it doesn't.
- **Relevant + new, not everything** — scope to the user's areas (feature-anchored, fuzzy) or where
  they're named as an owner; delta by when the release became known to us.
- **Config-driven, read-only on sources** — sources, feature tracker, window, triage, competitor subset, and
  match rule all come from config; nothing hardcoded.
- **Tie to your work** — resolve a related feature to an owned feature when possible and cross-link a
  competitor's move to the feature it pressures; relate across lenses, never hard-merge.
- **Every release deep-linked** — carry the release's source URL so the title links to it.
