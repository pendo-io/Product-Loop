# Member registry

The conductor's map of the skills it manages. It describes each member's **interface** (role, cadence, connectors, output, and which digest bucket its results map to), not its internal logic. Read this during setup (to decide what to offer) and on every run (to decide what is due and how to summarize).

When a new member skill is published, add a row here rather than hard-coding it elsewhere.

## Finding members (installed or not)

All members live in https://github.com/pendo-io/Product-Loop, one folder per skill. The names below are the public folder names at the time of writing. **Match by role, not only by name:** a member may be published or installed under a different name (for example a support-trends producer from another catalog). If a skill declares the same role and output (its description says it is a producer feeding the daily digest's 💬 lens from support tickets, say), treat it as that member. If two installed skills match one role, ask the user once which to use and store it in `config.members.<role>`.

At setup and at the start of each run:

1. **Check what's installed** (the client's skill list, plus `config.installed_skills`).
2. **Installed → use it.** Invoke the installed skill and pass it the shared inputs from config (below) so it doesn't re-onboard.
3. **Not installed → offer, never block.** Skip it this run. Add a 🟡 install offer **only on a day the member would have run** (its cadence, per the gating rule below; a weekly member is offered on its day, not every morning), unless muted. **Bundle all of that day's install offers into one 🟡 line** so they count as one proactive ask, for example *"4 of your enabled skills aren't installed yet (product-manager-weekly-update, …); want me to add them?"* A single missing member reads: *"feature-validation isn't installed yet; want me to add it so it can run each morning? (Say 'stop reminding me about this one' to mute.)"* If the client can install skills (a skill-install tool, or copying the folder into the client's skills directory when the user allows it), install on a yes; otherwise give the repo link and folder name. On mute, set `config.installed_skills.<name>.mute_reminder = true` and skip it silently.
4. **Record the choice** in `config.installed_skills.<name>` (`installed`, `version` if the skill declares `metadata.version`, `checked_at`, `mute_reminder`) so the digest can explain what ran.

**Passing shared config.** Pass each member the shared inputs from `config.json` (`user`, `role`, `sources` derived per `references/sources.md`, `focus_areas`, `org_constants`, review preferences, and any per-member config keys). A member's own config, if it has one, is still honored and the shared inputs fill gaps, except `org_constants`, which a member reads first for org-wide values (see the readers table under Org constants in `SKILL.md`). A member run by hand reads the same shared config from the config home when it can see it.

## Scheduled members: per-user (the conductor enables and schedules these)

Personalized per user and safe to run on the user's behalf.

| Member (public name) | Role it plays | Applies to | Native cadence | Connectors | Output | Default digest bucket |
| --- | --- | --- | --- | --- | --- | --- |
| `product-manager-weekly-update` | Weekly PM update writer | PM (also a PL who owns features) | Weekly, on your team's update day (set at setup) | Tracker, Slack, docs, Jira (all optional per the member) | A weekly update draft, published to the user's tracker if they chose that; review link if the review step is on | ✅ Done (published), or 🟡 if the review step is on |
| `product-lead-weekly-update` | Weekly product-lead rollup writer | Product Lead | Weekly, on your team's rollup day (set at setup), after the PM updates it reads | Tracker, Slack, docs, Jira | A rollup of the PMs' updates, checked against the PL's own context | ✅ Done (published); 🔴 for any PM-vs-context discrepancy it flags |
| `feature-validation` | Feature record validator | PM (their owned features) | Daily | Tracker or pasted list; Slack, Jira, docs optional | Stamps last-validated / validated-by; Tier C edits the validated fields only after approval (tiers are defined in `SKILL.md`); lists due features with an empty roadmap description and offers `feature-creation`'s Backfill mode | Tier A ✅ Done; Tier B/C and the backfill offer 🟡 Needs your OK |
| `feedback-idea-producer` (**producer**) | Feedback and ideas | PM/PL (their focus areas) | Daily (may be set weekly) | Pendo MCP (Listen); tracker optional | Delta-only feedback themes (typed bug / request / pain / praise, clustered, escalation-flagged); **proposes links** from feedback to confidently matched ideas and applies them after the user confirms (applies them without asking only when `feedback_idea.auto_link` is `true`; default `false`); **suggests** connecting each idea to its feature record | 💬; ✅ applied links; 🟡 confirm links / connect / create-idea / ambiguous; 🔴 a truly blocking escalation |
| `engineering-progress-producer` (**producer**) | Engineering progress | PM/PL (their boards + focus areas) | Daily (optional weekly team summary) | An issue tracker (Jira, GitHub Issues, Linear, or similar) or a pasted export; feature tracker optional | Delta shipped / blocked / newly broke (customer-facing first) the PM might have **missed**, each measured against the feature's description and goals, plus product-consequential decisions (`decisions`); reconciles `support_trends.eng_wait_queue`; read-only on the issue tracker; writes only its own config, plus the optional weekly team summary after the user approves it | 🚢 (primary, including `decisions`); 🔴 customer-facing break or an eng-wait escalation with no matching bug; ⚠️ Heads-up and ⚠️ Conflicts (conditional); 🔴 includes `needs_you` |
| `support-trends-producer` (**producer**) | Support trends | PM/PL (their focus areas) | Daily | A support desk (for example Zendesk) or a CSV export; tracker optional | Delta **support-ticket trends** across the PM's features, themes whose volume deviates from baseline, the PM might have **missed**; read-only on its sources, writes only its own config (stages actions) | 💬 (primary); 🟡 staged actions (content ticket / defect / link to an existing idea); account facts handed to the composer for ⚠️ |
| `field-qual-producer` (**producer**) | Field calls | PM/PL (their focus areas + watch accounts, the customer accounts the PM follows) | Daily | A call recorder (for example Gong); tracker optional | Recent calls' qualitative **field signal** on the PM's areas and the convergent cross-account themes they'd **miss**; marks the PM's own calls as attended (they count toward convergence, never surfaced as new); read-only on its sources, writes only its own config (stages actions) | 💬 (primary); 🟡 staged actions (link to an existing idea) |
| `competitor-release-producer` (**producer**) | Competitor releases | PM/PL (their focus areas) | Daily | The user's competitor-release tracker (a database or list), changelogs or feeds via web fetch, or pasted notes | Delta **competitor releases** relevant to the user, scoped to their areas; resolves a release to an owned feature for cross-linking; asks a yes/no relevance question for unclassified releases; **read-only**; its settings live in the shared `competitor_radar` block | 🧭 Competitive watch (conditional); 🟡 relevance questions |
| `digest-composer` (**composer**) | Digest composer | Everyone | Every run, **last** | none | The composed digest body | The digest body |

Before 1.1.0 the composer and the conductor had other names; `references/updates.md` → "Renamed skills" maps them, and the migration there renames them in a saved config. A skill installed under an old name that is now a deprecated stub never matches a role.

Producers fold into the composer's existing categories and add no section of their own, except the competitor-release producer, which feeds the conditional 🧭 lens.

## Scheduled members: optional (off until the user enables them)

| Member (public name) | Role it plays | Applies to | Native cadence | Connectors | Output | Default digest bucket |
| --- | --- | --- | --- | --- | --- | --- |
| `launch-watch` | Launch-health dashboard | PM/PL (the launches they name) | On demand by default; a weekly re-pin if the user enables it, stored as `weekly-<day>` and run by the conductor's daily task | Pendo MCP; no scheduler of its own here (the dashboard's Generate button runs in the page and needs no task) | Re-pins the same dashboard artifact with a new dated snapshot, archives the old one, and posts a one-line read | ✅ Handled (the dashboard link plus its one-line read); not composed into a lens |

### Cadence gating rule

On a given run, a member is "due" if:
- cadence is **daily** → always due;
- cadence is **weekly-<day>** → due only when today (real date, user tz) is that day. **Spread weekly members across the week so no single day gets heavy:** each weekly writer runs on the day set at setup (or the default when setup skipped it: the PM update on the last working day of the week, the PL rollup on the first; see `references/daily-run.md` step 2), a weekly producer on its assigned day, and `launch-watch`'s weekly re-pin on the day chosen when it was enabled. (The editor also trims, and a user can move any day.)
- **optional** members only run if the user enabled them; core members run whenever due.

## On-demand members + proactive gap checks (surface as suggestions)

User-triggered by nature, and never scheduled, except `launch-watch`, which can also run as the weekly re-pin above. During a run, if the conductor detects a relevant trigger, it adds a 🟡 suggestion line with the specific action.

| Member | Trigger to watch for during a run | Suggestion to surface |
| --- | --- | --- |
| `feature-creation` | The user mentions a feature that has no record yet, or a tracked item with no feature record; or `feature-validation` listed owned features with an empty roadmap description | "Want me to create the feature record? (Net-new works best from a conviction doc.)" For empty descriptions: "*{N} of your features* have no roadmap description; want me to draft them? (Nothing is written without your OK.)" |
| `product-announcement-generator` | A feature the user owns moved to GA / Open Beta / Early Access and has no announcement | "Want me to draft a product announcement? (Announces only what shipped.)" |
| `release-note-drafter` | An owned feature reached GA with no customer-facing "What's new" entry | "Want me to draft the release-note copy?" |
| *(engineering-team gap check)* | An owned feature in the tracker with an empty engineering team and/or engineering owner field (only when a tracker is connected and has those fields) | "*{Feature}* has no engineering team assigned; assign one?" If several lack one, offer a **single bulk assign** rather than one nag per feature. |
| `conviction-doc` | A bet or initiative is being scoped or debated that should have a conviction doc before build; or (exec roll-up) a conviction doc is not READY (the conviction-doc skill's verdict that a bet is ready to build) | "Want me to draft or stress-test the conviction doc?" |
| `prototype-fidelity` | The user shares a screenshot or asks to build, recreate, or mock up a UI | "Want me to build this as a pixel-accurate prototype?" |
| `launch-watch` | An owned feature recently reached GA / Open Beta, or the user asks how a launch is landing, and no launch dashboard exists yet | "Want me to stand up a launch-health dashboard? (Adoption, guide engagement, friction, and feedback sentiment for the features, pages, and guides you name.)" |

Do not fabricate triggers. Only surface an on-demand suggestion when a run genuinely surfaces the signal. These checks never write anything; they only produce suggestions. Skip a gap once resolved, and don't re-nag day after day.

The conductor creates or references conviction docs; it never creates PRDs.

## Tiers: core vs optional

Every member is either **core** (offered at first run, role-matched) or **optional** (offered later via the nudge in `references/optional-skills.md`). This keeps first-run setup focused while letting the catalog grow.

**Core:** `digest-composer`, `feature-validation`, `product-manager-weekly-update` (PM) / `product-lead-weekly-update` (PL), plus the **producers**: feedback and ideas, engineering progress, support trends, field calls, and competitor releases.

**Optional (offered later, never auto-enabled):**
- *Producers not yet published*: a launch-adoption monitor (emits a delta adoption line into 📈; distinct from `launch-watch`, which is a standalone dashboard, not a composer feed), analytics trends, and so on.
- *Schedulable optionals*: `launch-watch`, on demand by default with a weekly re-pin the user can enable (see "Scheduled members: optional" above). It isn't composed into a lens; its re-pin shows under ✅ Handled.
- *On-demand tools*: `release-note-drafter`, `product-announcement-generator`, `feature-creation`, `conviction-doc`, `prototype-fidelity`, and `launch-watch` when its weekly re-pin is off. Always available to invoke; "offering" them means introducing them or surfacing them when relevant.

## Role → default-enabled set (core only; optionals offered later)

Present the core set pre-selected in the setup checklist; the user adjusts. Confirm the role first.

- **Product Manager (PM)** → `feature-validation` (daily) and `product-manager-weekly-update` (weekly), plus the installed producers and `digest-composer`. Surface on-demand members as suggestions.
- **Product Lead (PL)** → `product-lead-weekly-update` (weekly), `feature-validation` for their own owned features, plus the installed producers and `digest-composer`. Add `product-manager-weekly-update` only if the PL personally owns features and wants a personal weekly update.
- **Exec** → a **rolled-up, mostly read** digest: summarize the latest PL and PM updates across product areas rather than writing personal updates. `conviction-doc` is especially relevant: the roll-up can show which initiatives have READY vs not-READY conviction docs.
- **Other** (any other product-team role) → the same per-user members scoped to their own owned features.

## Connector union

The setup connector check is the union across the *enabled* set. Every connector is optional at the conductor level; a member whose connector is missing is skipped with a 🔴 "connect X" line, or runs in its own no-connector mode if it has one.

- A **tracker** (Notion, Jira, Airtable, or similar) for writers that keep feature records, and for producers that anchor to features.
- **Slack** for members that read channels or deliver to Slack.
- **Google Drive / docs** for the weekly writers.
- **Jira** (or another issue tracker) for `engineering-progress-producer`, and Jira for the weekly writers and on-demand drafters that read Jira.
- **Pendo MCP** (usage, Listen feedback) for `feedback-idea-producer`, `launch-watch`, and `conviction-doc` evidence. Recommend connecting it when those are enabled; never block.
- A **support desk** (for example Zendesk) for `support-trends-producer` (it also accepts a CSV export).
- A **call recorder** (for example Gong) for `field-qual-producer`.
- **Calendar** for run-time inference during setup (nice to have; falls back to asking).
- **Email** only if an installed member needs it.

## Lifecycle index: answer "which skills help with {stage}?"

A generic five-stage product lifecycle. Map the user's own stage names onto it if they use different ones.

| Stage | What happens | Members |
|---|---|---|
| **Discover** | Find problems worth solving from customer, field, and market signal | `feedback-idea-producer`, `support-trends-producer`, `field-qual-producer`, `competitor-release-producer` |
| **Define** | Decide what to build and describe it | `conviction-doc`, `feature-creation`, `prototype-fidelity` |
| **Build** | Track delivery and keep feature records accurate | `engineering-progress-producer`, `feature-validation` |
| **Launch** | Tell customers and the field what shipped | `release-note-drafter`, `product-announcement-generator` |
| **Measure** | See how a release is landing and what to change | `launch-watch`, plus the Discover producers watching for post-release feedback and tickets |

`daily-digest`, `digest-composer`, and the weekly writers (`product-manager-weekly-update`, `product-lead-weekly-update`) span every stage rather than sitting in one.
