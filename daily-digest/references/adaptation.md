# Adaptation: learning how the user works and proposing changes

The conductor watches how each user actually uses it and, when it spots a stable pattern
(something the user clearly likes, or a repeated friction they keep working around), it proposes
a change. This is **built into the conductor** (not a separate skill) because it needs
first-class access to every run's outcome, the provenance-tracked config it would tune, and the
same digest it surfaces recommendations in. It's kept as a self-contained module (this file +
the `learning` config block) so it could be lifted out later if it ever needs to be shared.

Core stance: **propose, don't impose.** Learned changes are recommendations in the 🟡 bucket
until the user accepts them (with one opt-in exception below). Nothing about how the user works
is changed behind their back.

## What it observes (the observations log)

On every run, and when the user interacts with a digest, append lightweight, **behavioral**
observations to `learning.observations` (a rolling, capped log; store metadata, not message
content). Signals worth logging:

- **Off-cadence manual runs**: user runs a skill by hand often, outside its schedule (→ maybe the cadence is too slow).
- **Reschedules / "run later" / skipped-then-late**: the daily run keeps firing when the user isn't around (→ run time is wrong).
- **Suggestion dismissals**: the user repeatedly ignores or declines the same *kind* of 🟡 suggestion (→ mute it).
- **Consistent edits to outputs**: the user reliably edits a published update or brief the same way (→ a tone/format/length preference to fold into that member's inputs).
- **Repeated source adds of a kind**: e.g. always adding a customer-feedback channel for each new area (→ offer a standing rule).
- **Repeated gap-check snoozes on the same item**: e.g. always snoozing a gap-check nudge on one feature (→ offer to exclude it).
- **Ignored sections**: the user never engages with a given skill's digest output (→ offer to disable that skill).
- **Digest channel/timing tweaks**: repeated manual forwarding or re-timing (→ suggest a channel/time change).

Each observation: `{ date, signal, target (skill/field/source), detail }`. Cap the log at whichever
limit comes first, ~500 entries or ~90 days, and summarize older entries before trimming them.

## Detecting a pattern (don't jump on noise)

Only form a recommendation when a signal is **stable**: seen **≥3 times over ≥1 week** (tune per
signal), and not already dismissed. One-offs never trigger a proposal: that would be exactly
the noise this design is trying to avoid. Combine related observations into a single pattern (e.g.
five off-cadence runs of feature-validation → one "run it twice-daily?" pattern).

## Proposing a change

When a pattern qualifies, add a `learning.recommendations` entry (`id`, `pattern`, `proposal`,
`status: proposed`, `seen_count`, `first_seen`, `last_seen`) and surface **one** line in the
next digest's 🟡 bucket, phrased as an observation + a specific, reversible action:

- "You've added a customer-feedback channel for each new area you picked up: want me to add one automatically when your focus areas change?"
- "You haven't opened the support trends in your digest for a month: want me to stop running support-trends-producer?"
- "You forward the digest to your team channel most mornings: want me to post it there too?"
- "You rewrite the opening line of every weekly update draft into a single headline sentence: draft it that way by default?"

Cap learning suggestions at **1 to 2 per digest** so this never becomes noise itself.

## Applying an accepted change

On accept, write the config change and record provenance `set_by: learned` (with `base_value`
= the prior value and the recommendation `id`) so a later update treats it as a deliberate
user customization and never clobbers it. Mark the recommendation `accepted`. On decline, mark
`dismissed`, add the pattern to `learning.dismissed_patterns`, and **stop surfacing it** (a
dismissed pattern can resurface only if the behavior changes markedly later).

## Autopilot (opt-in)

If the user says something like "just tune it for me" / "you decide", set
`learning.autopilot: true`. In autopilot the conductor may **auto-apply low-risk, easily
reversible** changes without asking, limited to: run-time shifts within a sensible window,
cadence tweaks, muting a repeatedly-dismissed suggestion, and digest formatting. It then reports
them in the ✅ Done bucket ("moved your run to 8:45 after a run of late starts; tell me if you
want the old time back"). Autopilot **never** auto-changes anything that writes to a tracker or Slack
on the user's behalf, enables/disables a skill, or alters ownership/identity: those always stay
proposals. Default is `autopilot: false`.

## Guardrails

- **Propose by default; autopilot only when explicitly opted in, and only for low-risk tweaks.**
- **Respect dismissals**: never re-nag a dismissed pattern.
- **≤2 learning suggestions per digest**, and skip entirely on quiet days unless something is urgent.
- **Behavioral metadata only**: the observations log stores what happened (signal + target), not the content of messages, updates, or briefs.
- **Reversible**: every learned change is stated with how to undo it; keep the prior value in provenance.
- **Plays with updates**: learned changes look like user customizations (see `references/updates.md`), so a future update won't overwrite them; and if the user consistently rejects a spec's "you might want to try" flag, record that here so it isn't offered again.

## Config block

```json
"learning": {
  "enabled": true,
  "autopilot": false,
  "observations": [
    { "date": "<YYYY-MM-DD>", "signal": "reschedule", "target": "schedule.time", "detail": "asked for the digest later in the morning" }
  ],
  "recommendations": [
    { "id": "rec-001", "pattern": "reschedule:schedule.time x3", "proposal": "schedule.time 08:00->08:45", "status": "proposed", "seen_count": 3, "first_seen": "<YYYY-MM-DD>", "last_seen": "<YYYY-MM-DD>" }
  ],
  "dismissed_patterns": ["suggest:release-note-drafter"]
}
```
