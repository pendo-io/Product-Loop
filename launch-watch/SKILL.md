---
name: launch-watch
description: Weekly launch-health check for a Pendo customer's own recently launched feature (adoption curve, guide engagement, activation funnel, friction signals, and NPS / Listen sentiment), pulled live via Pendo MCP. Renders a shareable dashboard artifact where the surface supports HTML artifacts, a self-contained HTML file where it doesn't, and a text block otherwise. Use whenever a PM asks to check in on how a launch is landing, wants a "launch health" or "launch watch" view, asks "how is [feature] adopting," wants to know if a new guide is working, or asks to track a launch week over week. Always confirm which features, pages, and guides to track before pulling data, since this skill has no built-in registry of what a customer has launched. Single owner of the launch-health read for a named launch. Not for a product area's weekly health (use pulse) or diagnosing one funnel drop (use funnel-dropoff-investigator).
license: MIT. LICENSE has complete terms
compatibility: Needs the Pendo MCP connector for data (it is the skill's data source). Optional - Pendo Listen (feedback sentiment), a surface that renders HTML artifacts (dashboard; otherwise an HTML file or text block), scheduled tasks (weekly re-pin), and Notion (snapshot mirror, only if asked). No other skills required.
metadata:
  author: Pendo Product Operations
  version: "1.1.0"
---

# Launch Watch

A recurring health check for one or more recent feature launches, built on Pendo MCP analytics tools. Its best output is a **persistent dashboard artifact** the PM reopens week over week, not a one-time text digest. This skill is generic: it has no knowledge of what any given customer has shipped, so the first thing it always does is ask.

## Done

**Result:** one launch-health dashboard per run, covering every tracked launch: a published artifact where the surface supports it, else a self-contained HTML file, else the text block below. Plus a one-line read in chat.
**Done:** every section shows data, "no signal yet", or its unavailable caveat; every number links to its Pendo source, or links are left out because the host or subscription id is unknown; the "As of" date is set; on a re-pin the outgoing snapshot is archived and a row is appended to `history/index.json`; the one-line read ("landing well / narrow / with friction" plus the single reason) is posted in chat.
**Next consumer:** leadership and teammates who open the pinned artifact or its export; the re-pin task, which updates the same artifact; a daily digest or unattended run, which reads the one-line chat read.
**Intent:** catch a launch that looks adopted but is in trouble, not just report a number that went up (see "Why this shape").

## Boundaries and authority

- Never guess an ID, because a wrong ID silently reports on the wrong entity; resolve every name with `listCountables`, `listGuides`, or `searchEntities` first, bake the resolved IDs into the artifact, and if a name doesn't resolve, say so and offer the closest matches.
- Never manufacture a number, because this is a report people cite; a missing signal renders as "-" or "no signal yet", in the dashboard and the text alike.
- Never run the funnel stand-in or the proxy NPS without the user's yes, because neither is the real metric (the stand-in isn't sequence-verified, the proxy isn't real NPS movement); offer them, and on unattended runs mark the funnel unavailable with its caveat instead (Step 2).
- Never assume a Pendo host or subscription id, because the host differs by region; build links from `CONFIG` and leave them out when either is unknown.
- Never create a new artifact on a re-pin, because the share link must stay stable; update the same artifact, and if the surface can't update in place, produce a new dated file and say so.
- Never keep old numbers or an old read under a new date when a live refresh call fails, because the page would misstate what was measured when; keep the pinned snapshot and its original "As of" date and show "refresh failed: <which call>" (see "Sharing, refresh & archive").
- Never delete archived snapshots without explicit approval, and never the latest snapshot per launch, because the archive is the point-in-time record of what was reported; show the count and date range and ask (see "Quarterly cleanup").

**Authority:** this skill may publish the dashboard artifact and later update that same artifact in place, or write a self-contained HTML file; write `<workspace>/.launch-watch/history/<launch-key>/<YYYY-MM-DD>.html` and append a row (with the artifact URL) to `<workspace>/.launch-watch/history/index.json`; register a re-pin scheduled task, only if the user accepts at setup; run `buildPendoSegment`, only after the user says yes to the proxy NPS or a segment cut (its write semantics aren't documented, so treat it as a write); mirror snapshots to Notion, only if the user asks; and delete archived snapshots, only on explicit approval. Inside the page, a viewer's own clicks can store that viewer's own data in browser storage (a remembered tab, their own unpinned snapshots, never the pinned or shared data), call `HOST.ask` for a re-derived read, trigger the re-pin task through `HOST.runTask`, and download exports. Every other Pendo call is a read.

## Judgment vs mechanics

The model judges: scoping the launch with the user, which feedback items are really about the launch, the health level against the user's targets, the verdict, the narrative read, captions, per-guide and per-surface reads, and when to offer the deeper cuts. Pendo MCP tools compute the counts, rates, funnel conversion, retention curves, and feedback topics. The template's code owns the page structure, empty states, deep links, rate-unit formatting (`ratePct`), and, on a live refresh, the mapping from tool responses to sections, the dead-click percentage, the summary cards, and the stop on any failed call. Still done in prose by the model on the main run: the funnel stand-in ratio, the dead-click percentage, week-over-week deltas, the frustration-per-visitor trend, and the archive and `index.json` writes. No code gates the done check above.

## Honesty

- An empty result is valid: "no sentiment signal for this launch yet" is a legitimate finding, and if nothing looks off, say so plainly rather than manufacturing a watch item.
- Couldn't check is not nothing there: a tool that errors or times out is "unavailable right now", not "no signal"; a rate whose unit isn't stated is "unit unclear"; a failed live refresh shows "refresh failed: <which call>"; no Pendo MCP connected is said in one line.
- Proxies and small samples are labeled: the funnel stand-in "isn't sequence-verified"; proxy NPS is a proxy, always shown with its response count, and under roughly 20 responses is directional at best and stays out of the health summary (the 20 is a rule of thumb, not a statistical test); pasted numbers are labeled user-provided; a live view is labeled as not the pinned snapshot.
- Check each feedback item's `source` before counting it as independent signal, so a guide's own poll isn't counted as other feedback.
- Read back each write before reporting it done: re-open the published or updated artifact and check its "As of" date; re-read the archived snapshot file and the new `index.json` row; list scheduled tasks to confirm the re-pin task exists; re-fetch the Notion mirror page if one was written.

## Why this shape

A launch can look "adopted" and still be in trouble: a guide gets views but nobody finishes it, a feature gets clicks but they're mostly rage clicks, or usage is real but concentrated in three accounts. The point of this skill is to catch that, not just report a number that went up. The dashboard makes the *shape* of a launch legible at a glance, and can be refreshed so the PM watches the curve move, not a frozen snapshot.

## Setup (config)

Nothing here is hardcoded. Collect these once (from the connector's own context where possible, otherwise ask) and bake them into the dashboard's `CONFIG`:

| Value | Where it comes from |
|---|---|
| `subscriptionId` | The Pendo subscription the data belongs to. It's the id the Pendo MCP tools take for the subscription, and it appears in the Pendo app URL after `/s/`. |
| `pendoHost` | The host the user signs in to Pendo on, without `https://`. It differs by region, so copy it from the browser's address bar rather than assuming one. |
| `appId` | The app, if the subscription has more than one (`listAllApplications`). |
| `mcpPrefix` | The tool-name prefix the surface uses for the Pendo connector (for example `mcp__<server>__`), needed only for in-page refresh. |
| `targets` | Per launch, the launch targets the user sets (for example "20 new accounts a week by week 4", "guide completion 10%"). Ask at setup. Optional: with no target, the health read is not colored (see Step 1). |

If `subscriptionId` or `pendoHost` is missing, the dashboard still renders; it just leaves out the deep links.

**No Pendo MCP connected?** Say so in one line and point the user to connecting it (the rest of this skill assumes it). Offer to continue from numbers they paste or export from Pendo instead (usage, guide metrics, feedback): run the same analysis and output on that data, label every figure as user-provided, mark what's missing as "no signal yet," and skip in-page refresh.

## Step 1: Scope the launch (always ask, don't assume)

Before pulling anything, get from the user:

1. **What shipped.** One or more of: a page, a feature, a track event, a guide (walkthrough, tooltip, or banner announcing the launch), or a combination. Get names, then resolve to Pendo IDs with `listCountables` (pages/features/trackEvents) and `listGuides` (guides). If the user only has a vague description ("the new sync button"), use `listCountables` or `searchEntities` with `searchType='fuzzy'` or `'semantic'` to help them find it rather than guessing an ID.
2. **App**, if the subscription has more than one (`listAllApplications`).
3. **Launch date**, or "when did this go out". This anchors the before/after comparison. If they don't know exactly, a rough week is fine.
4. **Cadence**: one-off check or recurring weekly. If recurring and the surface supports scheduled tasks, this is a candidate for a scheduled run (see "Sharing, refresh & archive"); otherwise just run it once.
5. **Launch targets**: ask what the launch is meant to hit (adoption, guide completion, funnel conversion, or anything else the user tracks). Record each as a metric plus a target in the launch's `targets`. If the user has none, that's fine: say the health read will be a verdict without a green, amber, or red color, and continue. Don't suggest or invent default targets.
6. Optional: a **segment** to cut by (for example beta cohort vs. GA, a specific account tier) if adoption breadth is a concern. Don't ask this unless the user brings it up or the numbers below suggest it's worth checking.

Don't proceed to pulling data until you have at least #1. Everything else has reasonable defaults (all visitors, last 7 vs. prior 7 days, one-off, no targets). **Capture every resolved ID, the app, the launch date, the targets, and any segment: these get baked into the artifact so it can re-pull on its own.**

**Health color comes only from targets.** Green, amber, or red on the health read means "measured against the user's launch targets." With targets, judge the level against them and say which target drove it. Without any, set the level to `none`: the banner shows the verdict in a neutral style, and the read still says whether the launch is landing well, narrow, or with friction.

## Step 2: Pull the core signals

Run these for whatever mix of entities was named in Step 1. Skip any that don't apply (for example no guide was involved, or there's no NPS survey tied to this launch).

**Adoption curve**: `acquisitionTrend` (scope='feature'/'page', periodType='week') for first-time usage since launch, plus `aggregateEntityUsage` with `compareToDateRange` (current week vs. prior week, or since-launch vs. equivalent pre-launch window) for the trend on total events, unique visitors, and unique accounts.

**Guide engagement**: if a guide is in scope, `guideUsage` (views, completion rate, dismissal rate, time on guide) or `aggregateGuideMetrics` if there are several guides to compare. Use `guideEffectivenessMetrics` if the user cares about the dashboard-standard engagement and completion rates specifically. Guide rates (`completionRate`, `dismissalRate`) can come back as a fraction or a percent. Take the unit from the tool's response or its schema, never from the size of the value: if it says percent, show the value as a percent; if it says fraction, multiply by 100. If neither states the unit, show the number as returned, labeled "unit unclear", rather than guessing. For `guideUsage` the template's `rateUnit` defaults to `percent` (percent of total views); switch it to `fraction` if the tool's response or schema says so (see "How to build it").

**Activation funnel**: if both a guide and a feature/page are in scope, run `queryFunnel` from guide-seen to feature/page-used. This answers the sharper question underneath "guide engagement": of the people who saw the launch guide, how many actually went on to use the thing. A guide with high views but a weak funnel conversion means the announcement worked but the feature didn't land.

If `queryFunnel` errors or times out, don't just drop the metric. Tell the user it's unavailable right now, then **offer** a rougher stand-in: pull `entityUsage`/`aggregateEntityUsage` unique-visitor counts for the two steps separately (for example visitors who clicked "start" vs. visitors who completed "finish") and present the ratio as an approximate conversion. Flag clearly that this isn't sequence-verified the way a real funnel is (it's "how many touched each step," not "how many went from one to the next in order"), and only run it if the user wants that instead of waiting or retrying.

**Unattended runs** (a scheduled re-pin with no one to approve): don't run the stand-in. Mark the funnel unavailable, with the caveat "queryFunnel unavailable; stand-in not run (unattended run)", and say so in the one-line chat read. On the next attended run for this launch, if the current pin carries that caveat and `queryFunnel` still fails, offer the stand-in then.

**Friction signals**: pull the frustration fields already included in `entityUsage`/`aggregateEntityUsage` (rage clicks, error clicks, dead clicks; u-turns for pages). A launch with rising adoption *and* rising frustration-per-visitor is not a clean win. Flag it.

**Sentiment: check Listen feedback before reaching for a survey score.** Pull qualitative feedback tied to the launch first: `getFeedbackItems`/`getFeedbackInsights`/`generateFeedbackTopics` with `similaritySearchTerms` matching the launch name, or scoped to its `productAreaIds` if one exists. This catches feedback from every source (support tickets, sales and CS call transcripts, embedded polls), not just a guide's own survey, and it's usually more informative than a score this early in a launch. Check each item's `source` field (`PollResponse` is usually an in-app guide or poll, while `ZendeskTickets` and call-transcript sources are independent signal) so a guide's own poll doesn't get counted as "other feedback." A product-area filter returns everything tagged to that area, not only items about this launch, so hand-pick what's actually about the launch rather than reporting the whole batch.

**In the dashboard, lead the sentiment section with clustered themes, not raw titles.** Use `generateFeedbackTopics` (scoped to the launch's product area) for the "why" (theme name, description, and count), then show a few real **verbatims**: in-app poll responses (the title is the quote, with its 1 to 5 rating) and substantive form or ticket items (real descriptions). **Drop call-sourced feedback pointers**; show only items that carry a usable quote. Treat a call-sourced item as a pointer when it has fewer than 8 words once the opening pleasantry is removed (everything before the first clause that names the launch); the template's live refresh applies the same check. Real call quotes would need a call-transcript source, which most PMs won't have connected; don't fake them. Only after that, check whether there's a **dedicated survey** tied to the launch (`listSurveys`) and pull `surveyScores`/`surveyResponses` if one exists with responses. If neither the survey nor Listen turns up anything, **say so plainly rather than manufacturing a number**: "no sentiment signal for this launch yet" is a legitimate finding.

**Last resort:** if there's genuinely nothing else, **offer** (don't run unprompted) scoring the general product NPS survey for a segment of people who've used the launched feature (`buildPendoSegment` on the feature's `used` metric, then `surveyScores` with that `segmentPipeline`) against the unscoped baseline. This is a proxy, not real NPS movement. Say so, and only run it once the user says yes. Always report the response count alongside the score. Anything under about 20 responses is directional at best, not a number to put in the health summary. The 20 is a rule of thumb, not a statistical test.

## Step 3: Optional deeper cuts (offer, don't force)

These add real signal but need more setup. Offer them after the first pass rather than running them by default:

- **Retention and stickiness**: `cohortRetentionCurve` with `cohortMode='first'` on the launched feature/page. Tells you whether first-time users came back, that is, whether adoption is durable or a one-time try.
- **Segment comparison**: rerun the core signals with a `segmentPipeline` (built via `buildPendoSegment`) to check whether adoption is broad or concentrated in a handful of accounts.
- **Session replay sample**: if friction signals spike, `sessionReplayList` filtered to the feature/page with the relevant `frustrationTypes` gives a small qualitative sample of what's actually going wrong, without having to guess from the numbers alone.

## Output: a launch-health dashboard

Pick the richest form the current surface supports, in this order:

1. **Dashboard artifact.** If the surface can publish HTML artifacts that keep their own URL across sessions, build the dashboard there. It's a **pinned snapshot of record** that carries the FULL analysis (not a summary) and, where the surface allows the page to call connector tools, also supports an on-demand live refresh. It renders its **baked snapshot on open with no connector call**, so it can be shared with your team, including leadership who have no Pendo connector, and the numbers a viewer was shown never shift under them. It includes customer quotes, so check it before sharing it outside your team. How refreshing and re-pinning work is in **Sharing, refresh & archive** below.
2. **Self-contained HTML file.** If the surface can't publish artifacts but you can write files, build the same dashboard as a static HTML file with this run's numbers baked in (no live re-pull) wrapped in a full page (`<!doctype html>`, `<meta charset="utf-8">`, the template inside `<body>`), and hand it over, noting it's a snapshot. In a static file the Generate button only shows a hint to re-run the skill, and the snapshot hint says to re-run the skill to update (a local file has no artifact toolbar or live link); exports still work.
3. **Text block.** If HTML can't be rendered or written at all (a text-only context, or an unattended digest), use the compact text block below, which still carries every number and the read.

**Every number links back to its source in Pendo.** This is a report people cite, so make each element traceable: guides link to the guide (`/s/<subscriptionId>/guides/<guideId>`); friction surfaces and the adoption and usage figures link to the page, feature, or object detail (`/pages/<id>`, `/features/<id>`, `/objects/<id>`); each feedback verbatim links to its Listen item (`/s/<subscriptionId>/listen/validate?openItem=<feedbackId>`). Build these from `CONFIG.subscriptionId` + `CONFIG.pendoHost` (see Setup; never assume a host). Pendo's API does not expose a bespoke "report" URL for a metric, so the entity's own detail page, which shows that metric, is the honest source-of-record link. Anywhere a link is possible, include it. If the host or subscription id is unknown, omit links rather than guessing.

**The dashboard must preserve every layer of the analysis.** The failure mode to avoid is collapsing a rich read into a few single-number cards. Concretely, keep: each guide as its own row with its own views, visitors, completion, dismissal, and status; each friction surface (every page, feature, or action in scope) as its own row with visitors, dead-click %, rage clicks, and dwell time; the funnel with any caveat (for example "stand-in: counts per step, order not verified"); the adoption trend as a chart plus a caption that says what it's doing ("flat for three weeks, stepped up after the walkthrough"); and the analytical **narrative Read** in full, as prose (the same paragraph the text version would have written, not a one-liner). If the source analysis has a nuance ("most rage clicks on Reconnect follow an error message, so it reads as a failure rather than impatience"), the dashboard carries that nuance.

**How to build it:**

**Start from `references/dashboard-template.html`**, a detail-preserving skeleton whose `SNAPSHOT` data model already has slots for all of the above (banner verdict + full `narrative`, summary `cards`, `adoption` series + `caption`, a `guides[]` array, a `funnel` with `caveat`, a `friction.surfaces[]` array + `summary`, `themes[]` of `{name, description, count}`, and `sentiment[]`). Its `SNAPSHOT` and `HISTORY` ship with fictional demo data; always replace both. Fill its `CONFIG` with the values from Setup and Step 1 (`subscriptionId`, `pendoHost`, `appId`, `mcpPrefix`, the launch date, any segment, and `rateUnit`: `percent` by default for `guideUsage`, meaning completion and dismissal on a 0 to 100 scale of total views; use `fraction` only if the tool's response or schema says so, or leave it empty if you're using another tool that doesn't state a unit, so the page labels rates "unit unclear") and each launch's `entities` as `[{type:'page'|'feature'|'trackEvent'|'guide', id, name}]` (fetchLive uses these to re-pull on refresh: pages, features, and track events are each queried with their own type, guides through `guideUsage`; track events have no frustration counts, so their dead and rage cells show "-"). Replace the `SNAPSHOT` object with THIS run's full data, keeping every sub-metric, status, dwell, caveat, and read. If the surface lets the page call connector tools, ask Claude, or run a scheduled task, map those functions in the template's `HOST` block, and map each Pendo tool response into the `fetchLive()` stubs (probe each tool once in chat first to learn its real shape, then map around what you saw); declare every tool the page calls, the way the surface requires. If the surface has no such bridge, leave `HOST` empty: the page is then a static snapshot, which is fine. If the surface's artifact view already has its own reload control, don't add another.

**The page's structure (already laid out in the template):**
1. **Header**: feature name, launch date, "Week N since launch," and an "As of" date.
2. **Health banner**: one-line verdict, colored green, amber, or red only when the launch has targets (neutral otherwise), with the targets listed under it or a note that none are set.
3. **The read**: the full narrative paragraph, up top and prominent. This is the heart of the output; write it from the numbers and never drop it.
4. **Summary cards**: a row of at-a-glance figures with week-over-week deltas (direction colored by whether the move is good). These are a *summary*; the detail lives in the sections below. A card with no data renders "-", never a fabricated number.
5. **Adoption**: a weekly chart (Chart.js, with an inline-SVG fallback) plus the trend caption.
6. **Activation funnel**: the steps, the conversion %, and any caveat as a visible badge.
7. **Guide engagement**: one row per guide with its four metrics and a status pill; a one-line read under each.
8. **Friction**: one row per surface (visitors, dead-click %, rage clicks, u-turns for pages, avg time) plus the "where the friction isn't" summary note.
9. **Sentiment**: themes and hand-picked launch feedback with its `source` tag, or "no signal yet."
10. **Per-launch tabs**: if more than one launch item is tracked, each gets its own tab.

**Styling.** Clean, analytical, data-dense, in the style of a product-analytics adoption dashboard: white cards, soft rounded corners, subtle shadows, muted grays for secondary labels, one accent color (the template ships with pink; swap the CSS variables at the top of its `<style>` for your own brand), and green, amber, and red reserved for the health read (only when targets are set) and deltas.

**Persistence and storage.** Browser storage holds only that viewer's own conveniences and unpinned snapshots (a remembered tab, a browser-local *Generate* fallback snapshot), wrapped in try/catch, never the pinned or shared data. Keep any text baked into a raw `.html` file ASCII-safe (HTML entities or `\u` escapes) so it renders anywhere.

Whichever form it takes, **finish with a one-line read in chat** ("landing well / narrow / with friction" plus the single reason) so a daily digest or an unattended run stays legible without opening the artifact.

**Text fallback block** (per tracked launch item):

```
## [Feature/Page/Guide name] - launched [date]

**Adoption:** [trend in one line - e.g. "6 weeks in: slow start, stepped up after the wk4 walkthrough, 34 new accounts latest (111 total) - trending up"]
**Guide engagement:** [per guide: views / visitors / completion % / dismissal % + status]
**Activation funnel:** [step-to-step conversion %, with caveat if a proxy]
**Friction:** [per surface: visitors, dead-click %, rage clicks, dwell - flag the hotspot]
**NPS/sentiment:** [score + n + direction, or "no signal yet"]

**Targets:** [each target and where it stands, or "none set - health not colored"]
**Read:** [the full narrative - is this landing well, narrow, or with friction, and why]
```

Don't pad the dashboard (or the text) with prose where the number speaks for itself, but never drop a real finding to make it tidy. If nothing looks off, say so plainly rather than manufacturing a "watch item." If something is genuinely concerning (adoption declining, funnel weak, the launch guide no longer showing), the health banner and the read lead with that instead of burying it under green metrics.

## Sharing, refresh & archive

This is a report people cite, so it behaves like a pinned document, not a query that re-runs under the reader. What follows depends on what the surface supports; use what's there and skip the rest.

- **Pinned snapshot = the shareable record.** On open, the artifact renders only its baked-in data and analysis, stamped "As of <date>." No connector is called, so it opens identically for the PM, a teammate, or a VP who has never used Pendo, and what someone was shown stays put. This is what you send to leadership. The dashboard carries a split **Export** control: primary click is **PDF** (via the browser's Print > Save as PDF), and a caret opens **HTML** (self-contained file, chart baked to an image) and **PNG** (rasterized dashboard). The exported file is what you attach to a leadership email and what gets written to the archive; a live share link, if any, comes from the surface's own sharing controls, so the page does not reimplement it.
- **Refresh is explicit and connector-gated.** Where the page can call the Pendo connector (via `HOST`), the refresh control re-pulls the numbers (fetchLive maps acquisitionTrend to adoption, guideUsage to guides, entityUsage to friction, getFeedbackItems and generateFeedbackTopics to sentiment, and recomputes the summary cards) AND re-derives the analysis over those fresh numbers (via `HOST.ask`), so the *catch* (verdict, narrative read, per-guide and per-surface reads, captions) refreshes with the data, never going stale against new numbers. The live view is clearly labelled as not the pinned snapshot, and a viewer without the Pendo connector simply never triggers it and keeps the record. Refreshing numbers without re-deriving the judgment is the failure mode to avoid: the judgment is the point of this skill. So if any live call fails, or the read can't be re-derived (no `HOST.ask`, or it fails), the page keeps the pinned snapshot and its original "As of" date, shows a visible "refresh failed: <which call>" note, and doesn't re-run the analysis on partial data.
- **Re-pinning updates the shared version.** Running the skill again ("refresh my launch watch," or a weekly scheduled task) re-pulls, re-analyses, and **updates the same artifact in place** (same URL) with a new pinned snapshot and a bumped "As of" date, carrying the prior week's numbers for the week-over-week deltas. The share link is stable; its contents advance only when the owner re-pins, never mid-read for a viewer. If the surface can't update an artifact in place, produce a new dated file instead and say so.
- **In-dashboard controls and the re-pin model.** The dashboard carries a **Generate new pinned snapshot** button and a **History** dropdown. An in-page button can't rewrite the persisted artifact itself, so where the surface supports scheduled tasks, the durable-pin bridge is a re-pin task: offer to register one at setup (it's the button's engine, not just an auto-refresh). Who schedules the weekly refresh depends on how the skill runs: under daily-digest-conductor, the conductor schedules the weekly re-pin, so register no weekly task here (an on-demand task for the button is still fine); standalone, this skill registers its own task and, if the user accepts, bake its id into `CONFIG.repinTaskId`. If they decline, skip it; the fallback below still works. The task's prompt must update the SAME artifact (never create a new one), bump "As of", move the prior snapshot into `HISTORY`, and archive the outgoing one. Cadence is the only variable: **the default is a task with no cadence**, an on-demand-only setup (durable pin on click, never auto-runs); offer **weekly** as the option for a recurring check (the shared pin stays current on its own AND the button can trigger an immediate re-pin between weekly runs via `HOST.runTask`), and set it only if the user asks for it. When no task can be created, *Generate* falls back to a browser-local snapshot plus a "run 'refresh my launch watch'" nudge, and that manual run must also update the same artifact. Record the artifact's URL or id at first pin (in the workspace, see Archive) so every refresh (button, scheduled, or manual) targets the same one. *History* reads a baked `HISTORY` array of prior pins (last several inline so they render offline; older ones as archive links), appended on every re-pin. If the surface asks for tool permissions on scheduled runs, tell the user to run the task once by hand to approve its Pendo and artifact tools, so button-triggered runs don't pause on permissions.
- **Archive every pinned snapshot.** Before overwriting the pin, save the outgoing snapshot as a **dated, self-contained HTML file** (renders offline, no connector) under a history store, `<workspace>/.launch-watch/history/<launch-key>/<YYYY-MM-DD>.html`, and append a row to `<workspace>/.launch-watch/history/index.json` (`{launch, asOf, level, verdict, path, artifactUrl}`). That gives leadership a durable, point-in-time record of exactly what was reported each week, independent of the live artifact. If the surface can't write files, say so and offer the HTML export instead. If the user keeps their work in Notion, offer to also mirror each snapshot to a "Launch Watch Snapshots" page or database; don't assume it.

## Quarterly cleanup

The history store grows, so prune it, but never silently.

- On a **quarterly cadence** (or whenever the user says "clean up launch-watch history"), read `index.json` and check for snapshots older than the recommended cutoff.
- If there are any, **ask before deleting**, showing the count and the exact date range you propose to clear. **Recommended cutoff:** keep the most recent snapshot for every launch no matter its age, plus everything from the trailing two full quarters; recommend clearing snapshots dated before the start of the quarter two quarters back. State the recommendation and the kept and removed counts plainly, for example *"You have 23 launch-watch snapshots across 3 launches. Recommend clearing the 12 dated before the start of the quarter two quarters back (keeping the latest per launch + everything from the last two full quarters). Clear them?"*
- **Only delete on explicit approval**, never remove the latest snapshot per launch, and report what was removed and what was kept. Let the user override the range.

## Notes

- The artifact is the memory: with a live refresh or a re-pin, and `compareToDateRange` on each pull, week-over-week works without the skill keeping its own log. For a static snapshot instead, rely on `compareToDateRange` each run, or have the user keep a running log.
- ID resolution and the no-manufactured-numbers rule live in "Boundaries and authority".
