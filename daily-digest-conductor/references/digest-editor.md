# Digest editor: the conductor is an editor, not a logger

The digest's job is to protect the user's attention. Left unchecked, every member contributes its own output and the digest becomes a wall, which defeats the goal (less manual work *and* less mental load). This pass runs after all members and the composer have produced candidate items, and turns "a log of everything" into "the few things that matter," while keeping a complete record elsewhere so nothing is lost.

Run this in the daily run just before delivering the digest.

## 1. Gather + rank

Collect every candidate item from this run (composer findings, each member's ✅/🟡/🔴 results, gap checks, install offers, nudges) plus any **carryover** from yesterday (§4). Score each by **urgency × impact**:
- **Urgency**: is it time-critical (a weekly update due today, a bet about to go into build, an expiring beta, a re-auth blocking today's run)?
- **Impact**: how much does it matter (a published weekly update >> one routine auto-validation)?

Time-critical items are never silently deferred (§4).

## 2. Cap each lens (show the few, keep the rest)

The digest body is the composer's **static categories** (💬 From the field & feedback → 📈 Adoption & usage → 🚢 Shipped & in-flight) plus the **conditional 🧭 Competitive watch** and **⚠️ Conflicts** (shown only when they fire), followed by **✅ Handled for you**, the **🟡 Needs your OK** and **🔴 Needs you to act** surfaces, and the set-aside tally. Cap each; never dump a full list:

- **The lenses:** show the top few entries per lens by urgency×impact (`digest_budget.show_per_bucket`, default 3); a clean static lens is a **one-line reassurance, never omitted**. **One issue = one entry**: fold signal + escalation + action together. Within a lens, put FYI entries first, then heads-ups, then needs-action (an **invisible** order; the labels are never printed). **Cross-link** an entry to a related one in another lens, reciprocally. Summarize any low-confidence tail (for example fuzzy overlaps) as a count.
- **Action items live in their lens, at the end.** When the user is present, render each as a question (batched across lenses); the rest carry over (§4). End a lens on an action only when one genuinely exists.
- **✅ Handled for you:** each autonomous item as a **hyperlink to the artifact** plus a **short readable "what changed" note** (for example *"[Calendar Sync](url): validated; every checked field still matches its sources."*). Proof of work, not recap. More than ~5 → show the top few plus *"+{N} more; ask me to list them, or open the activity log."* No fake expand; the links are the affordance. (§3 keeps the full record and revert path.)
- **🟡 Needs your OK / 🔴 Needs you to act:** staged approvals (feature validation's Tier-B/C staged items, including Tier-C proposed edits, staged drafts, competitor relevance questions), proactive asks (gap checks, install offers, optional-skill and other offers), and only-you blockers (re-auth, missing connector, a product-lead update discrepancy). These are **not** discovery signal and are **not** folded into the lenses. 🟡 render as questions (batched). 🟡 has **two separate counts**: show the **top 3 to 5 staged approvals** (the rest carry over), and count proactive asks only against the **2 to 3 proactive-ask cap** below, never against the 3 to 5. 🔴 shown in full; **🔴 is never capped or dropped**.
- **Set-aside tally:** one closing line, *"Set aside N already-seen items,"* noting if a surfaced trend was drawn from them.

**Hard cap on proactive asks:** across *all* nudge and gap types combined (gap checks, install offers (all of the day's missing members in one bundled offer, which counts as one ask), optional-skill nudges, adaptation nudges, "want more?" offers), surface **at most 2 to 3 per digest** (`digest_budget.max_proactive_asks`, default 3), highest-value first. The rest wait and age/snooze per `references/optional-skills.md`. This is the single most important guard against 🟡 fatigue.

## 2a. Earned on-demand suggestions (conviction docs, prototypes)

Two on-demand members may be *offered* in the digest, but only when **earned**, never as standing noise, and always under the proactive-ask cap:

- **`conviction-doc`**: surface when a run turns up a bet or initiative being scoped or debated with no READY conviction doc. Offer to draft or stress-test it. For **exec roll-ups**, put the READY vs not-READY picture in the digest **body** (a read), not as an ask.
- **`prototype-fidelity`**: only when the user shares a screenshot or clearly asks to build or recreate a UI. Don't surface it unprompted.

Both cite the concrete signal that prompted the offer, respect snooze, and drop the offer once resolved.

## 3. Collapse-with-audit: never lose an autonomous change

Collapsing ✅ in the digest is only safe because every autonomous change is recorded in two places, both with a revert path:

1. **An audit note on the item itself**, where the target supports comments (a tracker comment, for example). Apply it to *every* member that writes on the user's behalf. Standard format:
   > 🤖 Daily digest · {member} · {date}: {what changed}. Was: "{previous}" → Now: "{new}". To undo, edit here or tell the conductor "revert this."
   Include the previous value whenever known so a revert is trivial. If the target has no comments, the activity log alone carries the record.
2. **The local activity log** (`activity-log.jsonl` next to `config.json`): an append-only record of **every** digest item this run, all buckets (not just writes), so the user has one complete "what did it do / surface" history and the adaptation module has data. Each row: `{ ts, run_id, bucket, skill, summary, artifact_url, prev_value, revert_hint, status }`. With no writable folder, say the log couldn't be kept this run.

The collapsed ✅ line notes both ("noted on each item; full record in the activity log"). Only autonomous **✅** items need the audit note; 🟡 (user approved) and 🔴 (user did it) are visible by definition but still go in the log.

**Revert command.** "Revert what you changed on {item}", "undo yesterday's changes", "roll back the {member} edit" → read the log and the item's note, show what changed, and undo it where reversible (restore `prev_value`); where not cleanly reversible, guide the user. Confirm before writing.

## 4. Carryover: held-back items go to TOMORROW, not the member's next run

Items that didn't make the cut (over-budget 🟡, lower-confidence composer findings, un-surfaced gaps and nudges) are stored in `config.carryover` and **surface at the top of the next day's digest**, prioritized above fresh items of equal weight. Don't make the user wait for a member's next scheduled run to see something it already found.

- **Priority-aware:** time-critical items (§1) are **never** carried over silently; they stay in-bucket today (bumping a lower-priority item) or escalate to 🔴.
- **Age out:** a carried item that keeps not getting addressed ages/snoozes (surfaced once, then quiet for **3 days** unless it escalates, after which it may surface again) rather than reappearing every day.

## 5. Resolve today's questions, then (only then) offer more

When the user is present, work the 🟡s to closure before offering anything else:

1. **Ask today's questions** as multiple choice (typically Approve / Dismiss / Defer / Other), using the client's question tool if it has one (Cowork: `ask_user_input_v0`), else numbered options in plain text.
2. **Follow up in conversation** on anything the buttons didn't resolve until **every** shown question is **resolved or explicitly deferred**.
3. **Only then**, make the capacity offer: *"You're through today's questions; want to tackle {N} held-back items now, or leave them for tomorrow?"* **Never** offer carryover while any of today's questions are still open, and never assume capacity. The adaptation module learns over time whether this user wants "everything" or "just the essentials."

On an **unattended** run there's no one to ask: deliver the static digest (links plus the questions as plain text) and run this flow when the user next opens it.

## What actually renders in a chat (no fake expands)

Only two affordances are dependable in a chat message: **hyperlinks** and an **interactive question tool** (where the client has one). A plain chat message has **no native "click to expand"**; never rely on one. For "the rest," link to the artifact or log, or invite a follow-up ("ask me to show the rest"). On a **tracker-page** digest, real toggle blocks may be used; in **Slack**, put detail in a thread. Match the affordance to the surface (`references/outputs.md`).

**Every actionable item carries a deep link to the thing it's about**: the staged feature edit, the Jira issue, the connection settings, the tracker record. In Slack, on a tracker page, or on an unattended run there is no live prompt, so the deep link IS the affordance. Never surface a 🟡, 🔴, or ✅ without a link to where it's resolved. If a member handed over no complete URL, cite the item by its provenance reference in plain text (as `attention-radar` does) rather than a partial link. The bracketed options in examples (`[ Review 2 ]`) depict the interactive question; the link beside them is what carries every other surface.

## 6. Quiet by default

Most weekdays should be short. If, after capping, an action bucket (✅ / 🟡 / 🔴) or a conditional lens is empty, omit it; a static lens keeps its one-line reassurance. If the whole run has nothing meaningful, honor the quiet path (one line, or nothing with `digest.quiet_days`). A short honest digest is the win; never pad to look busy.

**Brief but readable, not telegraphic.** Write each item as a short, natural sentence with enough connective tissue to read smoothly at a glance. "Validated 4 features and they're all still accurate" reads; "validated 4 ✓" does not. Aim for one comfortable line per item (a dozen-ish words), in the user's voice setting. The editor caps *how many* items appear; it doesn't strip each one down to fragments.

**The calibration examples in `attention-radar` are the length contract.** If a draft is materially longer than those examples, it's over budget: consolidate (one entry per issue), demote to carryover, or cut. Treat a wall-of-text digest as a bug, not thoroughness.

## Config this pass reads/writes

- `digest_budget`: per-bucket caps and the proactive-ask cap.
- `carryover`: the deferred queue (pulled at the top, pushed at the bottom of a run).
- `activity_log`: path to the local log.
- (with the adaptation module) the learned "how much to offer" preference.
