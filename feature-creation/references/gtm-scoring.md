---
name: gtm-scoring
description: 'Computes a go-to-market (GTM) launch score across four dimensions (Customer Experience Impact, Customer Adoption Impact, Competitive Advantage / Win-Rate, Enablement Effort), total out of 16, and maps it to a launch tier. Run it directly: "score this launch", "what GTM tier is this", "how big a launch is X". Also designed as a shared scoring module, so a conviction-doc skill can call it once per initiative (reviewed mode, with the full reference card) and a feature-creation skill can call it once per enhancement feature (automated mode, lighter touch) or, for a Net-new feature with no score yet, in reviewed mode, and every caller uses the same rubric. Callers may bundle an identical copy so they run standalone; never write a different version of this rubric. Single owner of the GTM launch rubric, its thresholds, and the suggested GTM actions. Not for writing launch content (use release-kit) or for writing the conviction doc itself (use conviction-doc).'
license: MIT. LICENSE has complete terms
compatibility: Standalone — no required connectors. Pendo MCP (adoption data) and Pendo Listen (customer feedback) are used opportunistically, if connected, to ground inferred scores in evidence.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

# GTM Scoring Skill

Produces one GTM score for whatever scope it is handed: a launch a user
describes directly, an initiative (when called from a conviction-doc
skill), or a single feature (when called from a feature-creation skill).

A **conviction doc** is a short pre-build statement of belief for an
initiative: the problem, who has it, the evidence, and what would prove
the bet wrong. "PM" below means whoever owns the product decision.

## Done

**Result:** one markdown table (Dimension | Score | Rationale | Suggested GTM actions) plus the literal total row, or the one line "Not scored: Light feature, release note optional."
**Done:** each of D1 to D4 has a score from 1 to 4 that the PM chose or that carries the ⚠️ Claude-inferred flag; the total equals the sum of the four; the tier matches the thresholds table; the D4 cell holds the tier actions and enablement actions from "Suggested GTM actions" and the D1 to D3 cells hold "–"; the total row uses the literal format below.
**Next consumer:** the calling skill's GTM section (a conviction-doc skill, or a feature-creation skill's checklist), which drops the table in without reformatting, or the user in direct use.
**Intent:** the same score always gives the same actions, and a missing source never blocks the score; when evidence is thin, infer and flag.

## Boundaries and authority

- Never score a Light feature (a small change that needs no launch treatment, such as a copy fix, as classified by the calling skill, or by this skill in direct use), because a Light change gets no launch treatment; return one line instead: "Not scored: Light feature, release note optional."
- Never recompute a net-new feature's score once a conviction doc has scored it, because the conviction doc computed it once in reviewed mode; the feature-creation skill copies it forward.
- Never present a score a human did not choose without flagging it ⚠️ Claude-inferred, because the PM has to be able to see and confirm every inferred score; infer, explain the reasoning in one line, and flag it.
- Never block on a missing evidence source, because every source is optional and the skill runs standalone; work from what the user or calling skill provides.
- Never rebuild or summarize the reference card when the file can be read, because the card must carry the rubric's exact wording; show `assets/reference-card.html` unchanged, and fall back to building it (or a markdown table) only when it can't be read or rendered.
- Never write a different version of this rubric, because every caller must score against the same one; callers bundle an identical copy instead.

**Authority:** Read-only: writes nothing. Output is a markdown table in chat; in reviewed mode the shipped reference card is shown in the conversation as an HTML artifact, unchanged.

## Judgment vs mechanics

The model judges each dimension's level, writes the one-line rationale, picks the targeted question for a "help" dimension, and, in direct use, decides whether a change is Light. The rest is fixed lookups from the tables in this file: the total (the sum of four levels), the tier (the thresholds table), and the suggested actions (from the tier and the D4 level). No script computes these yet; the model does the sum and the lookups in prose, so the Done check above is what catches a slip. The reference card is a shipped file passed through unchanged, and a sync check kept outside this folder, in the publishing workspace, checks that bundled copies of this skill stay byte-identical.

## Honesty

- "Not scored: Light feature, release note optional." is a complete, valid result.
- A missing or unconnected evidence source is not a reason to block or to guess silently; infer from what is available and flag the score ⚠️ Claude-inferred.
- Every score a human did not choose is labeled as inferred, with its reasoning in one line.

## The Four Dimensions

Rate each dimension from 1 to 4; the four ratings add up to a score out of 16, which maps to a launch tier (see the thresholds below the table).

| # | Dimension | Question | 1 | 2 | 3 | 4 |
|---|---|---|---|---|---|---|
| D1 | **Customer Experience Impact** | How much does this change what a user sees, understands, or does inside your product? | **Invisible** — Purely backend. No user-visible change — performance, infra, or data pipeline work. | **Subtle change** — Label, copy, tooltip, or calculation update, or a small addition such as a new button or export on an existing screen. Users may notice; most won't without being told. | **Meaningful update** — An existing feature, screen or workflow changes noticeably. Users will remark on it. | **Net-new feature** — Entirely new tool, screen, workflow, or interaction pattern. Users need to discover and adopt it. |
| D2 | **Customer Adoption Impact** | How much is this release expected to increase adoption or stickiness for this feature or product area? | **No behavior change** — Edge case, bug fix, or niche workflow. Doesn't change how users engage. | **Existing users only** — Some current users may utilize. Unlikely to draw anyone new to the product. | **Announcement-worthy** — Worth proactively telling all existing users about. Could deepen engagement or retention. | **Brings in new users** — Expected to expand who uses this feature or the feature's overall product area. |
| D3 | **Competitive Advantage / Win-Rate** | How much does this strengthen your competitive position or increase win rate for someone evaluating your product? | **No change** — Too minor to impact win rate or competitive edge. Won't come up in competitive conversations. | **Closing a known gap** — Matches what key competitors offer or removes an objection. Rarely the deciding factor in a deal. | **Clear competitive angle** — Distinct story that meaningfully improves win rate in deals where you were previously losing. GTM should know how to use it. | **Leading the market** — Few or no competitors have this. Expected to unlock new segments, reverse losses, or be frequently cited as the reason you won. |
| D4 | **Enablement Effort** | How much new enablement work does this require across the GTM org? | **No enablement lift** — Help-center update, release notes, product announcement sufficient. No demo, training, or messaging changes required. | **Just in time** — Impacts value story and demo slightly. Minor updates to existing enablement but no live field time required. Covered in your regular internal update. | **Standard feature training** — Need live time with some GTM segments. Feature impacts positioning/value or there's pricing nuance. Requires your enablement lead time (for example N weeks). | **Full GTM enablement** — Typical of larger launches. Requires your launch training, messaging updates, and a full demo environment and script. All GTM functions and leadership aligned. |

**Tier thresholds**, by total score:

| Total | Outcome (tier) | What it means |
|---|---|---|
| ≤7 | Release marketing only (Tier 3) | Help-center article, release note, product announcement. |
| 8–13 | Needs a conversation (potential Tier 2) | Flag to Enablement and Product Marketing. |
| 14–16 | Potential Tier 1 launch | Flag to Enablement and Product Marketing. |

Tier 1, Tier 2, and Tier 3 are launch tiers: Tier 1 is your largest,
fully coordinated launch, Tier 2 the middle tier, and Tier 3 release
marketing only. From 8 up, the score gives a potential tier, and the
conversation with Enablement and Product Marketing sets it. If your org uses different
tier or team names, swap them in here once and every caller inherits
them.

**Total row** (literal, one line, below the dimension table):
`**Total: [N]/16, [Outcome (tier)]**`, using the label from the table
above, for example "**Total: 13/16, Needs a conversation (potential Tier
2)**".

**Suggested GTM actions** come only from the tier and the D4 level, so
every run gives the same actions for the same score:
- Tier actions, from "What it means" above: Tier 3, help-center article,
  release note, and product announcement; potential Tier 2, those plus a
  conversation with Enablement and Product Marketing to set the tier;
  potential Tier 1, the Tier 2 actions (including the conversation to set
  the tier) plus flagging Enablement and Product Marketing for Tier 1
  launch planning.
- Enablement actions, from the D4 level's description: level 1, none
  beyond the tier actions; level 2, minor updates to the value story,
  demo, and existing enablement, and a mention in your regular internal
  update; level 3, live training with the affected GTM segments,
  scheduled within your enablement lead time; level 4, your launch
  training, messaging updates, and a full demo environment and script.

Put the tier actions and the enablement actions in the D4 row's
Suggested GTM actions cell, and "–" in the D1 to D3 cells.

## Reviewed mode (called by a conviction-doc skill once per initiative, or by a feature-creation skill for a Net-new feature with no score)

1. Before asking anything, present the static reference card —
   "GTM Launch Scoring — Reference Guide". It ships as
   `assets/reference-card.html`, relative to the skill folder. Read that file
   and show it to the user **as an HTML artifact, unchanged**: don't
   rebuild it from the table above, and don't summarize it. It has one
   light-grey header band per dimension (colored D1–D4 badge + dimension
   name + its question), then a row of 4 equal white cards below it, each
   a numbered circle (1–4) + bold level name + its description, in the
   exact wording above. A threshold legend (grey/amber/pink dot + label)
   closes the card. Plain borders, no per-score color shading — the badge
   color is the only color signal, and it's per-dimension, not per-score.
   Non-interactive; the PM keeps it on screen while answering. If you
   change the rubric wording above, change the card to match. If the
   file can't be read, build the card to this description. If the
   surface can't render an HTML artifact or buttons (for example a
   terminal), show the same content as a markdown table and ask for a
   number (1–4) or "help".
2. Ask each dimension one at a time via single-select buttons: the 4
   scoring levels plus "Help me figure this out". Do not advance until
   answered, because each dimension needs a level the PM chose or a
   flagged inference before the total is summed (see Done).
3. For any "help me figure this out": if the evidence says little or
   nothing about that dimension, first ask one targeted question for it.
   One question per "help"
   dimension, asked before moving to the next dimension, for example:
   - D1: "Will users see a new screen or workflow, a noticeable change to
     an existing one, or only a small addition or label change?"
   - D2: "Who do you expect to use this: some current users, all current
     users, or people who don't use this area yet?"
   - D3: "Do competitors already offer this, and has its absence come up
     in deals you lost?"
   - D4: "Will the field need live training or your full launch
     training, or is a release note and announcement enough?"
   If the user answers "help" again or doesn't know, infer anyway.
   Then infer from the answer (if you asked) plus the conviction doc's
   evidence or the research the caller passes (problem, who has it,
   adoption data, customer-feedback themes), or from whatever the user provided in direct use, explain
   the reasoning in one line, and flag it ⚠️
   Claude-inferred in the output — never silently guess.
4. Calculate the total and apply the threshold.

**Return**: a markdown table — Dimension | Score (dot bar, e.g. ⚫⚫⚫⚪
3/4 — label) | Rationale | Suggested GTM actions (see "Suggested GTM
actions" above) — plus the literal total row. The calling skill drops this straight into its GTM
section; do not reformat it.

## Automated mode (called by a feature-creation skill, once per enhancement feature)

Same four dimensions, same thresholds — every enhancement gets its own
score, scoped to that feature, never inherited from the parent
initiative's conviction score. Lighter weight than reviewed mode:
- Skip the full reference-card render — infer from the feature's own
  description plus whatever parent-initiative context the caller passes,
  pulling analytics/feedback data opportunistically but not blocking on it.
- Still ask a quick single-select per dimension if the caller wants PM
  input; otherwise infer all four. Flag every score a human didn't choose
  as ⚠️ Claude-inferred. Reviewed mode's targeted questions don't apply;
  the caller has the PM confirm inferred scores.
- Return the same dimension table format and total row.

## Direct use (no calling skill)

If a user invokes this skill directly, run reviewed mode against whatever
they describe or paste (a spec, a launch brief, a one-line description).
With no calling skill, decide yourself whether the change is Light (as
defined in "Boundaries and authority" above) and, if so, say so rather
than running the card. "Help me figure
this out" still means infer and flag, with the same targeted question
first when the evidence is thin (reviewed mode step 3).

## Evidence sources (optional)

Pendo MCP and Pendo Listen are named here; any product-analytics or
feedback connector works the same way. If Pendo MCP is connected, use it
for current adoption and usage of the feature or product area (D2). If
Pendo Listen or another feedback tool is connected, use feedback themes
and request volume (D2, D3). Without either, work from what the user or
calling skill provides. Never block on a missing source.

## What this skill never does

See "Boundaries and authority" above: no score for a Light feature, no recomputing a conviction doc's score, and no score without the ⚠️ Claude-inferred flag when a human did not choose it.
