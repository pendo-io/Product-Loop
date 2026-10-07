---
name: funnel-dropoff-investigator
description: >
  Investigate funnel drop-off by combining quantitative funnel analysis, session replay sampling,
  and qualitative feedback into a single diagnostic report. Use this skill ANY TIME someone wants
  to understand where users are dropping out of a flow, why conversion is low, or what's blocking
  users from completing a key journey. Trigger on phrases like "why are users dropping off",
  "investigate funnel", "where are users getting stuck", "analyse drop-off", "funnel conversion",
  "show me replays for drop-off", "what's causing low conversion", "pull replays for the worst step",
  or any request to understand a user flow or journey breakdown. Also trigger when someone shares
  a funnel name or page sequence and asks what's going wrong.
---

# Funnel Drop-off Investigator

Diagnose why users drop out of key flows by chaining three data sources — funnel metrics, session
replays, and qualitative feedback — into a single actionable report. This is the fastest way to
move from "conversion is low" to "here's why, and here's evidence."

## Workflow

### Step 1: Gather Context

**Ask the user:**
- Which funnel or user journey to investigate (e.g., "Trial signup", "Booking confirmation", "Onboarding")
- Time period (default: last 30 days)
- Product area or page names if known

**Get app context:**
```
Use list_all_applications to confirm subId and appId
```

---

### Step 2: Run the Funnel Analysis

**Find the funnel by name or search for relevant pages:**
```
Use searchEntities with:
- itemType: "Page" (and optionally "Feature")
- query: [user-described flow name]
- limit: 20
```

If the user has a named Pendo funnel, use `queryFunnel` directly.
If they're describing a journey rather than an existing funnel, construct the step sequence
from the pages/features found above and confirm with the user before running.

**Run the funnel:**
```
Use queryFunnel with:
- subId, appId
- steps: [ordered array of pageId/featureId]
- startDate / endDate
- Returns: visitors at each step, conversion rate per step, time between steps
```

**Identify the critical drop-off:**
- Find the step with the largest absolute visitor drop
- Calculate the step-level conversion rate for each transition
- Flag any step where conversion < 50% or drops >20% compared to the previous step

**Present a step summary to the user before continuing:**
```
Step 1 → Step 2: X visitors → Y visitors (Z% conversion)
Step 2 → Step 3: Y visitors → W visitors (V% conversion) ← ⚠️ Biggest drop
Step 3 → Step 4: ...
```

---

### Step 3: Pull Session Replays for the Critical Step

Target the step immediately **before** the biggest drop-off point — these are the sessions
where users reached the problematic moment but didn't convert.

```
Use sessionReplayList with:
- subId, appId
- startDate / endDate
- pageId: [page ID of the critical drop-off step]
- minDuration: 10 (seconds — filter out accidental bounces)
- activityPercentile: 20 (focus on lower-engagement sessions — the confused or stuck ones)
- limit: 5
```

Present the replays as a numbered list with:
- Session ID / visitor ID
- Duration on page
- Activity level
- Direct link to replay (if URL available from result)

> **Note to Claude:** Don't summarise what's in the replays — you can't watch them. Simply surface
> them with enough context for the PM to click through and watch the 3–5 most informative sessions.
> Explain what to look for: rage clicks, hesitation, repeated scrolling, form abandonment.

---

### Step 4: Pull Qualitative Feedback for the Drop-off Area

```
Use get_feedback_items with:
- subId (Listen subscription)
- dateRange: last 30 days
- limit: 50
```

Then filter/score feedback relevant to the funnel area:
- Match on page name, feature name, or flow keywords
- Look for sentiment patterns: confusion, friction, missing info, error states
- Prioritise verbatims that mention the specific step or page name

Surface the top 5–8 most relevant feedback items as direct quotes with source/date.

**Also run topic clustering if volume warrants it:**
```
Use generate_feedback_topics with:
- same filters as above
- Returns: AI-clustered themes with representative quotes
```

---

### Step 5: Generate the Investigation Report

Produce a Markdown artifact saved to `/mnt/user-data/outputs/funnel-investigation-[flow-name]-[date].md`

#### Report Structure

```markdown
# Funnel Drop-off Investigation: [Flow Name]
**Date:** [today]  **Period:** [date range]  **App:** [app name]

## TL;DR
[2–3 sentence summary: where drop-off is worst, what the replays suggest, what feedback confirms]

## Funnel Overview

| Step | Visitors | Conversion | Drop |
|------|----------|------------|------|
| Step 1: [Name] | X | — | — |
| Step 2: [Name] | Y | Z% | -N |
| ⚠️ Step 3: [Name] | W | V% | -M ← Critical |
| Step 4: [Name] | ... | | |

**Critical drop-off:** Step 2 → Step 3 loses the most users (N visitors, V% conversion)

## Session Replays — What to Watch

These 5 sessions show users who reached [critical step] but didn't complete it.
Look for: rage clicks, repeated scrolling, hesitation before CTA, form errors.

1. [Visitor/Session ID] — [X]s on page, [activity level] — [link or ID]
2. ...

**What to look for in these sessions:**
- [Specific UI element or CTA name] — did users find it?
- [Form field or input] — were there errors or abandonment?
- [Navigation pattern] — did users go back or exit?

## Qualitative Feedback — What Users Are Saying

**Top themes from [N] feedback items in this area:**

### Theme 1: [Theme name] ([N] mentions)
> "[Representative verbatim quote]" — [Source, Date]
> "[Second quote if strong]"

### Theme 2: [Theme name] ([N] mentions)
> "[Representative verbatim quote]"

[Repeat for 3–5 themes]

## Hypotheses

Based on the funnel data, replays, and feedback, the most likely causes of drop-off are:

1. **[Hypothesis 1]** — Supported by: [data point / feedback theme]
2. **[Hypothesis 2]** — Supported by: [data point / feedback theme]
3. **[Hypothesis 3]** — Supported by: [data point / feedback theme]

## Recommended Next Steps

| Action | Priority | Owner |
|--------|----------|-------|
| [e.g. Watch 5 replays at step X] | High | PM |
| [e.g. Add tooltip/guide at step X] | High | Design |
| [e.g. A/B test CTA copy] | Medium | PM/Eng |
| [e.g. Add NPS/feedback prompt at drop-off step] | Medium | PM |
```

---

### Step 6: Present the Report

Use `present_files` to share the Markdown artifact.

Summarise in chat:
- The single biggest drop-off step and its conversion rate
- The #1 replay signal to investigate
- The dominant feedback theme
- Your top hypothesis for what's causing the drop

---

## Pendo MCP Tool Reference

### queryFunnel
- `subId`, `appId`: required
- `steps`: ordered array of `{ type: "page"|"feature", id: "..." }`
- `startDate` / `endDate`: YYYY-MM-DD
- Returns: visitors per step, conversion rates, median time between steps

### sessionReplayList
- `subId`, `appId`: required
- `pageId`: filter to a specific page (the drop-off step)
- `startDate` / `endDate`
- `minDuration`: seconds (filter accidental bounces — use 10)
- `activityPercentile`: 0–100 (lower = less active = potentially confused users)
- `limit`: number of replays to return (5 is usually enough)
- Returns: session IDs, visitor IDs, duration, activity %, replay URL if available

### get_feedback_items
- `subId`: Listen subscription ID
- `dateRange` or `startDate`/`endDate`
- `limit`: number of items (50 is a good starting point)
- Returns: raw feedback verbatims with source, date, sentiment

### generate_feedback_topics
- Same filters as `get_feedback_items`
- Returns: AI-clustered themes with representative quotes and counts

### searchEntities
- `itemType`: "Page", "Feature"
- `query`: semantic search term
- `itemIds`: resolve IDs to names

### list_all_applications
- Returns available subscriptions and apps with IDs

---

## Error Handling

- **Funnel has no data**: Check date range — widen if needed. Confirm page IDs are correct.
- **No replays returned**: Widen date range, lower `minDuration`, or raise `activityPercentile`.
- **No relevant feedback**: Run without page filter to get broader feedback pool, then manually filter for relevance.
- **Can't resolve page names**: Show IDs and ask user to confirm which pages map to which funnel steps.
- **Funnel steps are unclear**: Present candidate pages from `searchEntities` and ask user to confirm the sequence before running.
