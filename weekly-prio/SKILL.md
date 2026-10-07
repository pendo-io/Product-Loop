---
name: "weekly-prio"
description: "Pull, cluster, and prioritise recent product feedback from Pendo Listen, then link the top themes directly to existing ideas in your product backlog. Use this skill for weekly or daily product prioritisation rituals, feedback triage, backlog grooming informed by qual data, or any time you want to know \"what are customers telling us right now?\" Trigger on phrases like \"weekly prioritisation\", \"triage my feedback\", \"what's customers are saying this week\", \"pull recent feedback\", \"link feedback to ideas\", \"what should I prioritise\", \"feedback themes\", \"cluster my feedback\", \"what are the top themes\", \"morning product review\", \"daily feedback digest\", or any request to review, sort, or act on recent customer feedback. Also trigger when someone asks what to work on next or wants to ground their backlog in customer evidence."
license: MIT. LICENSE has complete terms
---

# Weekly Prio

Pull recent customer feedback from Pendo Listen, cluster it into actionable themes, score themes
by volume and recency, link the top items to ideas in your product backlog, and run a targeted
competitive intel sweep framed by your product conviction document. Run this every Monday morning
(or whenever you need to ground your priorities in real customer evidence).

## Workflow

### Step 1: Gather Context

**Ask the user (if not already known):**
- Product area / label to focus on (e.g., "Reporting", "Calendar Sync", "Onboarding") — or "all"
- Time window (default: last 7 days; suggest 14 days if volume is likely low)

**First run only: ask for your direct competitors.** Read `config.json` in the working folder
first. If `weekly_prio.competitors` is missing, ask once: "Which direct competitors should the
weekly competitive sweep cover? Name two to five, or say none." Save the confirmed names to
`weekly_prio.competitors` (a list of names; an empty list means none) and leave other sections
of the file untouched. Later runs read the saved list and don't ask again unless the user says
"change my competitors". On a scheduled or unattended run, never ask: use what is saved.

**Get subscription context:**
```
Use list_all_applications to confirm subId
```

---

### Step 2: Pull Raw Feedback

```
Use get_feedback_items with:
- subId: [Listen subscription]
- startDate: [N days ago]
- endDate: [today]
- labels: [product area label if specified]
- limit: 100
```

**Report to user:** "Pulled [N] feedback items from the last [X] days."

If fewer than 10 items, suggest widening to 14 or 30 days before continuing.

---

### Step 3: Cluster Into Themes

```
Use generate_feedback_topics with:
- same filters as above
- Returns: AI-clustered themes with representative quotes, counts, sentiment
```

**Post-process:**
- Sort by volume (most-mentioned first)
- Annotate each theme with item count, dominant sentiment, and product area mapping

**Confirm themes with user:**
> "I've identified [N] themes. Here's a quick summary — let me know if any should be merged,
> split, or ignored before I go looking for matching ideas."

```
1. [Theme name] — [N] mentions, [sentiment]
2. [Theme name] — [N] mentions, [sentiment]
...
```

---

### Step 4: Surface Insights

For each top theme (top 5):
- Pull 2–3 verbatim representative quotes
- Note accounts or segments mentioned repeatedly (enterprise, power users)
- Flag urgent issues (outages, blockers, compliance, churn signals)

```
Use get_feedback_insights with same filters for a higher-level AI summary
```

---

### Step 5: Match Themes to Ideas

For each of the top 3–5 themes, search the product backlog:

```
Use get_ideas with:
- query: [theme name or keywords]
- limit: 5 per theme
```

For each theme, confirm the best match with the user:
> "For theme '[Theme Name]', I found these ideas — which should I link to? Or create a new one?"

---

### Step 6: Link Feedback to Ideas

For each confirmed theme → idea match:

```
Use link_idea_and_feedback with:
- ideaId: [confirmed idea ID]
- feedbackId: [feedback item ID]
```

Link the most representative 2–3 items per theme. Report linkage summary:
```
✅ Theme: [Name] → Idea: [Title] (linked 3 feedback items)
⏭️  Theme: [Name] → No match found, skipped
```

---

### Step 7: Generate the Prioritisation Brief

Save a Markdown artifact to `/mnt/user-data/outputs/prioritisation-brief-[date].md`

#### Report Structure

```markdown
# Weekly Prioritisation Brief
**Date:** [today]  **Period:** [date range]  **Label:** [label or "All"]
**Feedback volume:** [N] items pulled, [N] themes identified

---

## Top Themes This Week

### 🔴 1. [Theme Name] — [N] mentions ([sentiment])

**Summary:** [1–2 sentence synthesis]

**Representative quotes:**
> "[Quote 1]" — [Source/Account], [Date]
> "[Quote 2]" — [Source], [Date]

**Linked to:** [Idea title] ✅ (or "No match — suggest creating idea")
**Urgency flag:** [Yes/No — note if any item signals a blocker or escalation]

---

[Repeat for top 5 themes]

---

## Linkage Summary

| Theme | Idea Linked | Action |
|-------|-------------|--------|
| [Theme] | [Idea title] | Link N items |
| [Theme] | — none — | Create idea |

---

## Suggested Actions

1. [Highest urgency item] — why + owner
2. [Create new idea for theme X] — N mentions, no current idea
3. [Follow up with account Y] — mentioned repeatedly in theme Z

---

## Items Flagged for Escalation

[List any urgency signals: outages, compliance, churn risk, enterprise blockers]
If none: "No escalation-level items this period."
```

Use `present_files` to share the Markdown brief with the user.

---

### Step 8: Competitive Intel (run every week, immediately after the brief)

The competitive intel pass uses the themes from this week's brief as a lens — the goal is to
understand what direct competitors are doing on the same problems your customers are surfacing,
and to flag anything that links to your existing ideas or represents a net-new strategic signal.

Before 8a, check that `weekly_prio.competitors` has at least one name; if not, follow "If no
competitors are saved" in 8b.

#### 8a: Fetch the Product Conviction Document

Search Notion for the conviction document for this feature area:

```
Use notion-search with query: "[product area] conviction" OR "[product area] product brief"
```

If found in Notion, read the full document. If the document is in Google Drive (Notion will
show a link), extract the file ID and use `read_file_content` to fetch the body.

From the conviction doc, extract:
- **Core technical bet** (what your product is uniquely doing)
- **Key differentiators** (what competitors can't easily replicate)
- **Out-of-scope items** (things explicitly deferred — these are worth checking whether
  competitors have shipped them in the meantime)
- **Conviction stage** (Exploring / Committing / Executing)

If no conviction doc exists for this area, skip to 8b and use the top themes as the framing
instead.

#### 8b: Competitive Searches

For each direct competitor, run a targeted search framed by the conviction doc's bets.
The point isn't a generic "what is Competitor A doing" sweep — it's "what is Competitor A doing
on the specific problem our conviction doc is trying to solve?"

**Competitors to cover every week:** the names saved in `weekly_prio.competitors` (Step 1).
"Competitor A" and "Competitor B" in this file are only examples; never search for them.

**If no competitors are saved:** on an interactive run, ask the Step 1 question now. On a
scheduled or unattended run, skip Step 8 and add one line to the brief and the chat summary:
"Competitive sweep skipped: no competitors saved yet. Run weekly-prio once by hand to set them."
If the saved list is empty because the user said none, skip Step 8 and say "Competitive sweep
skipped: no competitors configured."

For each competitor, search for:
```
"[Competitor] [conviction doc core bet keyword] 2025 OR 2026"
"[Competitor] [product area] release OR launch OR update"
```

**Emerging players — check weekly:**
- YC recent batches: search "YC [product area] [year]" and "site:ycombinator.com [product area]"
- Product Hunt recent launches: search "site:producthunt.com [product area] [current year]"

#### 8c: Synthesise Competitive Findings

For each competitor and emerging player, produce a card with:

1. **What they shipped** — factual, one paragraph
2. **Threat level** — 🔴 HIGH / 🟡 MEDIUM / 🟢 LOW / 👀 WATCH
   - HIGH: directly addresses the same customer pain your conviction doc is solving
   - MEDIUM: adjacent — solves part of the problem or for a different buyer
   - LOW: different audience or meaningfully weaker approach
   - WATCH: early-stage or YC-funded, not a current threat but on the same trajectory
3. **Links to existing ideas** — does this move strengthen or undermine an idea in your backlog?
   If yes, call it out explicitly and suggest updating the idea description to reference it.
4. **Net-new strategic signal** — anything the competitor is doing that your conviction doc
   didn't anticipate and that you should be thinking about now

**Key framing:** always connect competitor moves back to the conviction doc's bets. "Competitor B
launched X" is useless without "...which directly competes with our bet that Y, because Z."

#### 8d: Update the Right-Panel Artifact

If a Cowork artifact exists for this feature area's prioritisation brief, add or refresh a
"🔍 Compete" tab to it with the competitive findings. Each competitor gets an expandable card
with threat badge, what they shipped, insight callouts, and idea linkage or net-new flags.

At the bottom of the Compete tab, include a "Net-new things to think about" section for any
strategic signals not in the conviction doc.

```
Use update_artifact with:
- id: [existing artifact id]
- html_path: [updated HTML file with compete tab added]
- update_summary: "Refreshed 🔍 Compete tab: [top competitor move] ..."
```

If no artifact exists, include the competitive findings as an appendix in the Markdown brief.

---

### Step 9: Present & Summarise

Summarise in chat:
- The #1 feedback theme by volume and its sentiment
- Whether any urgent/escalation items were found
- How many ideas were successfully linked, and how many gaps remain
- The top competitive signal from this week's sweep and its threat level, or the reason the
  sweep was skipped

---

## Pendo MCP Tool Reference

| Tool | Purpose |
|------|---------|
| `list_all_applications` | Confirm subId |
| `get_feedback_items` | Pull raw feedback verbatims |
| `generate_feedback_topics` | AI-clustered themes with quotes + counts |
| `get_feedback_insights` | Higher-level AI insight summary |
| `get_ideas` | Search backlog for matching ideas |
| `link_idea_and_feedback` | Attach feedback to an idea |

**For conviction doc:** `notion-search` → `notion-fetch` or Google Drive `read_file_content`

**For competitive intel:** `WebSearch` (one search per competitor per week)

---

## Tips for Best Results

- **Run weekly** — the value compounds when you compare themes week-over-week
- **Widen to 14 days** if a single week returns fewer than 20 items
- **Don't over-link** — 2–3 strong representative items per idea beats bulk-linking
- **Escalation first** — scan for urgency before prioritising by volume
- **Conviction doc is the framing** — competitive searches without it produce noise; with it they
  produce signal. If the conviction doc is stale, flag that to the user too.
- **Threat level escalates fast** — if a competitor ships something you scoped out-of-bounds,
  that's worth flagging even if the conviction stage says "Executing"

---

## Error Handling

- **Low feedback volume**: Widen date range; note in report
- **No conviction doc found**: Use top themes as framing; suggest the user create a conviction doc
- **No matching ideas**: Surface the theme; suggest creating a new idea
- **link_idea_and_feedback fails**: Note in report with IDs for manual linking
- **Competitor search returns nothing relevant**: Note "no material moves this week" for that
  competitor — don't skip or hallucinate
- **Themes look too broad**: Ask user to refine the product area filter and re-run clustering

