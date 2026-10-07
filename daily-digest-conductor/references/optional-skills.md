# Optional skills: the catalog and the nudge

Core members are offered at first run (see `references/member-registry.md` → "Tiers"). **Optional** members are not; they're introduced over time, so first-run setup stays focused and users adopt the rest when they're actually useful. The same nudge also covers **core members that aren't installed yet**.

## What the optional catalog holds

- **Schedulable optionals** (each one you enable is an extra run, and it may need extra connectors):
  - `launch-watch`: a launch-health dashboard (adoption, guide engagement, friction, feedback sentiment) for a recently launched feature (needs Pendo MCP plus the feature, page, and guide ids the user names). On demand by default; a weekly refresh can be enabled, which the conductor runs from its daily task (below). It is not composed into a digest lens; on its refresh day the re-pin shows under ✅ Handled as the dashboard link plus its one-line read.
  - Producers published after this version (for example a launch-adoption monitor).
- **On-demand tools** (always available to invoke; "offering" = introducing them, or surfacing when a run detects the trigger): `release-note-drafter`, `product-announcement-generator`, `feature-creation`, `conviction-doc`, `prototype-fidelity`, `launch-watch`.

Track state in `config.optional_skills`: for each, `status` (`offered` / `enabled` / `dismissed`), `last_offered` (date), and a global `stop_asking` flag.

## When to nudge (signal first, with a 2-week backstop)

Offer optionals in the digest's 🟡 bucket when **either**:

1. **A signal shows one would have helped** (the primary trigger): a run detects the user doing by hand, or missing, something an optional member does. For example a GA feature with no release note (→ `release-note-drafter`). Offer *that specific member*, not the whole list.
2. **The 2-week backstop**: when no signal has come up for ≥14 days since the last offer, do a single gentle check-in listing the still-unenabled optionals. Stay silent between backstops when there's no signal.

Never exceed the backstop cadence on top of signal-driven offers: at most one "here's the rest of the catalog" nudge per 2 weeks, plus targeted signal-driven ones as they genuinely arise. Respect `stop_asking`.

## Tie in what happened since the last ask

When offering (especially signal-driven), **cite the concrete activity** since the last offer that the member would have handled:

> "Since we last talked about this, two of your features reached GA with no release note; `release-note-drafter` would draft those from the Jira epic. Want to turn it on?"

Pull these from `learning.observations` and `last_run` history. If there's no concrete activity to cite, keep the offer short and factual rather than inventing urgency.

## Aging / snooze: surface once, then go quiet

An offer (or a gap check) that the user neither accepts nor dismisses should **not** reappear every day. Surface it once, then snooze it for **3 days** (`snooze`) before offering again, unless it **escalates** (the related bet gets closer to build, the friction signal spikes), in which case it may resurface sooner. This applies to optional offers, install offers, gap checks, and adaptation nudges alike. Combined with the editor's hard cap of 2 to 3 proactive asks per digest, this keeps "helpful" from becoming "naggy."

## Always offer the off-switch

Every nudge ends with an easy opt-out: *"…or say the word and I'll stop offering these."* On decline, mark that member `dismissed`. If they ask to stop entirely, set `config.optional_skills.stop_asking = true` (they can still enable any optional by asking). A `dismissed` member can resurface only if a strong new signal appears much later.

## Setting one up when accepted

- **Not installed** → install it (if the client can) or give the repo link, then continue below once it's there.
- **Schedulable optional** → walk through its part of onboarding (capture its config, for example the Pendo subscription id and product-area ids), verify its connectors, add it to `enabled_skills`. Mark `status: enabled`. For `launch-watch`'s weekly refresh, ask which day and store `cadence: weekly-<day>`; if the user skips, pick a day with no other weekly member and say which. The conductor's daily task runs the re-pin on that day, so `launch-watch` doesn't register a weekly task of its own (it may still register its on-demand re-pin task for the dashboard's Generate button). Run on its own, without the conductor, `launch-watch` registers its own weekly task instead.
- **On-demand tool** → nothing to schedule; confirm the user knows how to invoke it (or run it now if that's what prompted the offer). Mark `status: enabled` (introduced).

Enabling or dismissing is a user choice; record it with `set_by: user` provenance so an update never re-enables a dismissed member.
