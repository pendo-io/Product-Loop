# pulse

A quick read on how a product area is doing right now. Visitors, accounts, feedback, and frustration signals — all in one view, no dashboard required.

## When to use

- Monday morning before planning or standup
- Quick check before a leadership conversation
- After a quiet period when you want to know what moved

## Usage

```
/pulse
/pulse <product area>
```

If no product area is given, ask the user which area to focus on before proceeding.

---

## Workflow

### Step 1: Get app context

```
Use listAllApplications to get subId and appId
```

### Step 2: Find the relevant pages and features

Use `searchEntities` to find pages and features related to the product area:

```
searchEntities:
  itemType: "Page" then "Feature"
  query: <product area>
  limit: 10 each
```

Keep the top 5 pages and top 5 features by relevance.

### Step 3: Visitors and accounts — last 7 days vs prior 7 days

For each of the top pages and features, call `entityUsage` twice — once for the current period and once for the prior period:

```
entityUsage:
  period 1: today minus 7 days → today
  period 2: today minus 14 days → today minus 7 days
  metrics: visitors, accounts, totalEvents
```

Calculate the delta (absolute and percentage) for visitors and accounts between the two periods.

Flag anything that moved more than 10% in either direction.

### Step 4: Recent feedback

```
getFeedbackItems:
  dateRange: last 7 days
  limit: 20
```

If there are 5 or more items, run `generateFeedbackTopics` to cluster them. Surface the top 3 themes with a count of how many items fall into each.

If fewer than 5 items, list them individually with a one-line summary of each.

### Step 5: Frustration signals

```
listTrackedIssues:
  filter: rage clicks, errors, u-turns
  dateRange: last 7 days
```

Surface any issues that appear more than twice. For each, note the page or feature it's on and the number of affected visitors.

If no tracked issues exist, call `sessionReplayList` with rage_click or error filters and surface the top 3 sessions by severity.

---

## Output format

Keep it tight. The whole digest should be readable in under 2 minutes.

```
## Pulse — [Product Area] — [Date]

### Usage (last 7 days vs prior 7 days)
- Visitors: X (▲/▼ Y%)
- Accounts: X (▲/▼ Y%)
- Notable movements: [any page/feature that moved >10%]

### Feedback themes
1. [Theme] — X mentions
2. [Theme] — X mentions
3. [Theme] — X mentions

### Frustration signals
- [Issue type] on [page/feature] — X visitors affected
- [Issue type] on [page/feature] — X visitors affected

### One thing to watch
[One sentence — the most important signal from this pulse that warrants a closer look]
```

End with a one-sentence "one thing to watch" — the signal most likely to need action this week.
