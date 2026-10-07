---
name: launch-watch
description: Weekly launch health update for a Pendo customer's own recently-launched feature — adoption curve, guide engagement, activation funnel, friction signals, and NPS movement, pulled live via Pendo MCP. Use whenever a PM asks to check in on how a launch is landing, wants a "launch health" or "launch watch" update, asks "how is [feature] adopting," wants to know if a new guide is working, or asks to track a launch week over week. Always confirm which features, pages, and guides to track before pulling data — this skill has no built-in registry of what a customer has launched.
---

# Launch Watch

A recurring health check for one or more recent feature launches, built entirely on Pendo MCP analytics tools. This skill is generic — it has no knowledge of what any given customer has shipped, so the first thing it always does is ask.

## Why this shape

A launch can look "adopted" and still be in trouble: a guide gets views but nobody finishes it, a feature gets clicks but they're mostly rage clicks, or usage is real but concentrated in three accounts. The point of this skill is to catch that — not just report a number that went up.

## Step 1 — Scope the launch (always ask, don't assume)

Before pulling anything, get from the user:

1. **What shipped.** One or more of: a page, a feature, a track event, a guide (walkthrough/tooltip/banner announcing the launch), or a combination. Get names, then resolve to Pendo IDs with `listCountables` (pages/features/trackEvents) and `listGuides` (guides). If the user only has a vague description ("the new export button"), use `listCountables` or `searchEntities` with `searchType='fuzzy'` or `'semantic'` to help them find it rather than guessing an ID.
2. **App**, if the subscription has more than one (`listAllApplications`).
3. **Launch date**, or "when did this go out" — this anchors the before/after comparison. If they don't know exactly, a rough week is fine.
4. **Cadence** — one-off check or recurring weekly. If recurring, this is a candidate for a scheduled/Cowork-style run; otherwise just run it once.
5. Optional: a **segment** to cut by (e.g. beta cohort vs. GA, a specific account tier) if adoption breadth is a concern. Don't ask this unless the user brings it up or the numbers below suggest it's worth checking.

Don't proceed to pulling data until you have at least #1. Everything else has reasonable defaults (all visitors, last 7 vs. prior 7 days, one-off).

## Step 2 — Pull the core signals

Run these for whatever mix of entities was named in Step 1. Skip any that don't apply (e.g. no guide was involved, or there's no NPS survey tied to this launch).

**Adoption curve** — `acquisitionTrend` (scope='feature'/'page', periodType='week') for first-time usage since launch, plus `aggregateEntityUsage` with `compareToDateRange` (current week vs. prior week, or since-launch vs. equivalent pre-launch window) for the trend on total events/unique visitors/unique accounts.

**Guide engagement** — if a guide is in scope, `guideUsage` (views, completion rate, dismissal rate, time on guide) or `aggregateGuideMetrics` if there are several guides to compare. Use `guideEffectivenessMetrics` if the user cares about the dashboard-standard engagement/completion rates specifically.

**Activation funnel** — if both a guide and a feature/page are in scope, run `queryFunnel` from guide-seen to feature/page-used. This answers the sharper question underneath "guide engagement": of the people who saw the launch guide, how many actually went on to use the thing. A guide with high views but a weak funnel conversion means the announcement worked but the feature didn't land.

If `queryFunnel` errors or times out (it can), don't just drop the metric — tell the user it's unavailable right now, then **offer** a rougher stand-in: pull `entityUsage`/`aggregateEntityUsage` unique-visitor counts for the two steps separately (e.g. visitors who clicked "start" vs. visitors who completed "finish") and present the ratio as an approximate conversion. Flag clearly that this isn't sequence-verified the way a real funnel is — it's "how many touched each step," not "how many went from one to the next in order" — and only run it if the user wants that instead of waiting or retrying.

**Friction signals** — pull the frustration fields already included in `entityUsage`/`aggregateEntityUsage` (rage clicks, error clicks, dead clicks; u-turns for pages). A launch with rising adoption *and* rising frustration-per-visitor is not a clean win — flag it.

**Sentiment — check Listen feedback before reaching for a survey score.** Pull qualitative feedback tied to the launch first: `getFeedbackItems`/`getFeedbackInsights`/`generateFeedbackTopics` with `similaritySearchTerms` matching the launch name, or scoped to its `productAreaIds` if one exists. This catches feedback from every source — support tickets, sales/CS call transcripts, embedded polls — not just a guide's own survey, and it's usually more informative than a score this early in a launch. Check the `source` field on each item (`PollResponse` usually means an in-app guide/poll; `ZendeskTickets` and call-transcript sources are genuinely independent signal) so a guide's own poll doesn't get counted as "other feedback." Product-area scoping is often loose (a long call gets tagged with every topic it touched) — hand-pick what's actually about the launch rather than reporting the whole batch.

Only after that, check whether there's a **dedicated survey** tied to the launch (`listSurveys`) and pull `surveyScores`/`surveyResponses` if one exists with responses. If neither the survey nor Listen turns up anything, **say so plainly rather than manufacturing a number** — "no sentiment signal for this launch yet" is a legitimate finding.

**Last resort:** if there's genuinely nothing else, **offer** — don't run unprompted — scoring the general product NPS survey for a segment of people who've used the launched feature (`buildPendoSegment` on the feature's `used` metric, then `surveyScores` with that `segmentPipeline`) against the unscoped baseline. This is a proxy, not real NPS movement — say so, and only run it once the user says yes. Always report the response count alongside the score, and treat anything under roughly 20 responses as directional at best, not a number to put in the health summary.

## Step 3 — Optional deeper cuts (offer, don't force)

These add real signal but need more setup — offer them after the first pass rather than running them by default:

- **Retention/stickiness** — `cohortRetentionCurve` with `cohortMode='first'` on the launched feature/page. Tells you whether first-time users came back, i.e. whether adoption is durable or a one-time try.
- **Segment comparison** — rerun the core signals with a `segmentPipeline` (built via `buildPendoSegment`) to check whether adoption is broad or concentrated in a handful of accounts.
- **Session replay sample** — if friction signals spike, `sessionReplayList` filtered to the feature/page with the relevant `frustrationTypes` gives a small qualitative sample of what's actually going wrong, without having to guess from the numbers alone.

## Output format

Keep it scannable — this is a status check, not a report. For each tracked launch item:

```
## [Feature/Page/Guide name] — launched [date]

**Adoption:** [trend line in one sentence — e.g. "34 new accounts this week, up from 21 last week"]
**Guide engagement:** [views / completion % / dismissal %, if applicable]
**Activation funnel:** [guide-seen → feature-used conversion %, if applicable]
**Friction:** [flag only if notable — e.g. "rage clicks up 3x since launch, concentrated on the confirm step"]
**NPS/sentiment:** [score + direction, if applicable]

**Read:** [one or two sentences — is this landing well, landing narrow, or landing with friction]
```

Don't pad every section with prose if the number speaks for itself. If nothing looks off, say so plainly rather than manufacturing a "watch item." If something is genuinely concerning (adoption flat, friction rising, funnel conversion low), lead with that instead of burying it under a wall of green metrics.

## Notes

- This skill has no memory of past runs unless the user's environment provides one (e.g. a Cowork scheduled task or saved conversation). For a true week-over-week series, either rely on `compareToDateRange` each time, or have the user (or their own tooling) keep a running log.
- Resolve every name to an ID before calling analytics tools — `listCountables`/`listGuides`/`searchEntities` first, always. Never guess an ID.
- If the user names a feature that doesn't resolve, say so and offer the closest matches rather than silently picking one.
