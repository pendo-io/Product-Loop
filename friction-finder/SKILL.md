---
name: friction-finder
description: >
  Find where friction lives in a Pendo application by combining rage clicks, error clicks,
  dead clicks, and u-turns (page-level) with recent feedback flagged as a bug/product issue
  into a single ranked friction report. Use this skill ANY TIME someone wants to know where
  users are struggling, hitting bugs, or getting frustrated in an app — even if they only
  name one signal (e.g. "just show me rage clicks"). Trigger on phrases like "friction
  finder", "bug radar", "where's the friction", "where are users struggling", "find the
  friction", "rage clicks this month", "u-turns", "dead clicks", "frustration signals",
  "what's broken in my app", "show me bugs from feedback", "top pain points", "where is my
  app frustrating users", or any request to scan an app for usability/bug friction. Make sure
  to use this skill whenever click-frustration or bug-feedback data would answer the
  question, even if the user doesn't use the words "friction" or "bug" explicitly.
---

# Friction Finder

Rank the pages and features in an application by real frustration signal (rage clicks, error
clicks, dead clicks, u-turns), cross-reference with recent feedback tagged as a bug/product
issue, and pull a few session replays as evidence — producing a single "here's where the
friction is, and why" report.

## When to use

- A PM wants a general sweep of "what's frustrating users" without a specific hypothesis
- Prepping for sprint planning / bug triage and want data-backed candidates
- Someone asks about rage clicks, dead clicks, error clicks, or u-turns for an app
- Cross-referencing frustration signals against reported bugs to confirm they're the same problem

This is the general-purpose "where does it hurt" sweep. If the user already knows the specific
funnel/flow that's broken, prefer `funnel-dropoff-investigator` instead — that skill is for a
single named journey. Friction Finder is for scanning an entire app with no prior hypothesis.

## Usage

```
/friction-finder <app or product area>
/friction-finder <app> last 14 days
```

If no app/product area is named, ask which one before proceeding. Default lookback window is
**30 days** when the user doesn't specify one.

**Output is always a plain-text friction report**, delivered directly in chat — not an HTML
file, dashboard, or other visual artifact. This is a fixed property of the skill, not a
per-run stylistic choice.

---

## Workflow

### Step 1: Get app context

Confirm `subId` and `appId` — use `listAllApplications`, or resolve via `searchEntities` /
prior conversation context if the user already named an app. If the user named a product area
rather than an app, use `searchEntities` (itemType: "Page", "Feature") to scope down to that
area's entities in Step 2.

### Step 2: Rank pages and features by frustration signal

Use `aggregateEntityUsage` — it returns per-entity frustration counts
(`totalRageClickCount`, `totalErrorClickCount`, `totalDeadClickCount`, and, for pages only,
`totalUTurnCount`) alongside normal usage. Run it once per signal so the true top offenders
surface (a single totalEvents-sorted call can bury a high-rage-click, low-traffic page):

```
aggregateEntityUsage:
  entityType: "page"
  dateRange: last 30 days (or user-specified)
  sortBy: "totalRageClickCount"   → then rerun with "totalErrorClickCount", "totalDeadClickCount", "totalUTurnCount"
  sortOrder: "desc"
  limit: 10
  appId: <appId>

aggregateEntityUsage:
  entityType: "feature"
  same dateRange, appId
  sortBy: "totalRageClickCount"   → then rerun with "totalErrorClickCount", "totalDeadClickCount"
  sortOrder: "desc"
  limit: 10
```

That's up to 7 calls (4 for pages, 3 for features — features have no u-turn metric). Merge results
into one table per entity, keeping every frustration count even if the entity was only surfaced
under one signal's sort.

**Compute a friction score per entity** to rank overall (weights are a judgment call, not exact):

```
frictionScore = (rageClicks × 3) + (errorClicks × 2) + (deadClicks × 1) + (uTurns × 1)
```

Sort all entities by `frictionScore` descending. Take the top 5–8 as the "hot spots."

### Step 3: Pull recent feedback flagged as a bug

```
Use getFeedbackInsights (or generateFeedbackTopics if volume is high) with:
  filters:
    feedbackTypes: ["Product Issues"]
    appIds: [<appId>]
    startDate / endDate: same window as Step 2
```

Match feedback items against the hot-spot pages/features from Step 2 by name or keyword —
flag any overlap explicitly ("this page's rage clicks are corroborated by N bug reports").
Also surface bug feedback that *doesn't* match any hot-spot entity — that's a friction signal
Step 2 wouldn't have caught on its own (e.g. a backend/data bug with no distinct click pattern).

If there are 5+ matching feedback items for a single hot spot, run `generateFeedbackTopics`
scoped to that entity to cluster them into a theme rather than listing every item.

### Step 4: Pull evidence replays for the top 2–3 hot spots

```
Use sessionReplayList with:
  subId, appId
  startDate / endDate: same window
  pageIds / featureIds: [top hot-spot entity ID]
  frustrationTypes: [{ frustrationType: "rageClick", fact: "occurred" }]  (swap in errorClick/deadClick
    to match whichever signal is dominant for that entity)
  minDuration: 10000  (10s — filter out accidental sessions)
  limit: 3
```

> **Note:** Don't claim to have watched the replay — surface session ID, duration, and a direct
> link so the PM can review it themselves.

### Step 5: Generate the Friction Report — plain text, delivered in chat

**The output is a plain-text report, written directly into the chat response.** No HTML file,
no visual dashboard, no artifact of any kind — just clearly organized text using simple
Markdown (headers, bullets, bold) for readability. Don't build or reference any HTML template
for this step.

**Structure the report using the data from Steps 2–4:**

1. **Header** — app name and date range covered.
2. **Top finding** — the #1 ranked entity by friction score. State its score, name, the metrics
   that make it severe, and whether feedback corroborates it. Lead with the most concrete,
   specific fact available.
3. **Friction ranking** — a list or simple table of the top 6–8 entities from Step 2, sorted by
   friction score descending, with their rage/error/dead-click (and u-turn, for pages) counts.
4. **Feedback correlation** — state plainly whether recent bug-flagged feedback corroborates
   any of the ranked hot spots. If no matching feedback was found, say so explicitly — that's a
   data point in itself, not an omission.
5. **Hot spot deep dives** — a short write-up for each of the top 2–3 hot spots from Step 2:
   dominant signal (rage/error/dead-click), a one-sentence hypothesis, and its replay links
   from Step 3.
6. **Suggested next step** — one concrete, actionable recommendation based on the top finding.

### Step 6: Present the report

Deliver the report as the chat response itself. Keep it lean — the person should be able to
scan it in a few seconds and get to the #1 hot spot, whether feedback corroborates it, and the
suggested next step without digging.

---

## Pendo MCP Tool Reference

### aggregateEntityUsage
- `entityType`: "page" or "feature"
- `dateRange`, `appId`, `subId`
- `sortBy`: "totalRageClickCount" | "totalErrorClickCount" | "totalDeadClickCount" | "totalUTurnCount" (pages only) | "totalEvents" | etc.
- Returns per-entity: `totalEvents`, `uniqueVisitors`, `uniqueAccounts`, and (pages/features)
  `totalErrorClickCount`, `totalRageClickCount`, `totalDeadClickCount`; pages also
  `totalUTurnCount`, `avgTimePerVisitor`

### getFeedbackInsights / generateFeedbackTopics
- `filters.feedbackTypes`: use `["Product Issues"]` to scope to bug-flagged feedback
- `filters.appIds`, `filters.startDate`/`endDate`
- `generateFeedbackTopics` for clustering when volume is high (5+ items)

### sessionReplayList
- `pageIds` / `featureIds`: scope to the hot-spot entity
- `frustrationTypes`: `[{ frustrationType: "rageClick"|"errorClick"|"deadClick"|"uTurn", fact: "occurred" }]`
- `minDuration`, `limit`

### searchEntities / listCountables
- Resolve a named product area or app to page/feature IDs before ranking

---

## Error Handling

- **No entities returned for a sort key**: that signal may genuinely be at zero across the app —
  note it and move on, don't treat it as a tool failure.
- **No matching bug feedback**: say so explicitly in the report rather than omitting the section;
  absence of feedback is itself a data point (either it's not being reported, or it's not visible yet).
- **No replays returned for a hot spot**: widen the date range or drop `minDuration` before
  concluding there's no evidence available.
- **Ambiguous app/product area name**: present candidate apps/pages from `searchEntities` and
  confirm before running the full sweep.
