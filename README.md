# Product-Loop

**Your product is already telling you what to build next.**

The teams that win aren't the ones who hear it first — they're the ones who act on it in the same breath. Product-Loop puts a Claude skill at every turn of that loop: discover what's worth solving, validate before committing, launch with confidence, measure what landed.

Signal to decision to impact. Hours, not weeks.

Every skill is backed by live data from your Pendo subscription.

Works with Claude Code and Cowork.

---

## Install

```bash
git clone https://github.com/pendo-io/Product-Loop.git ~/.claude/skills/product-loop
```

Then authenticate Pendo once:

```
/mcp
```

That's it.

---

## Skills

### Monitor
Keep a constant pulse on what's happening.

| Skill | What it does |
|---|---|
| [`pulse`](./pulse/) | Weekly product health digest — usage trends, feedback themes, guide performance |
| [`weekly-prio`](./weekly-prio/) | Pull, cluster, and prioritise recent feedback from Pendo Listen — link top themes to backlog ideas |
| [`bug-radar`](./bug-radar/) | Daily scan for emerging bugs and errors before your users report them (coming soon) |
| [`usage-watch`](./usage-watch/) | Daily usage checker — flag drops, spikes, and anomalies in key features (coming soon) |

### Uncover
Find the real problems worth solving.

| Skill | What it does |
|---|---|
| [`funnel-dropoff-investigator`](./funnel-dropoff-investigator/) | Diagnose where users abandon a flow — funnel metrics, session replays, and qual feedback in one report |
| [`friction-finder`](./friction-finder/) | Blends rage clicks, drop-off, and qual feedback to surface where users are stuck |
| [`theme-scan`](./theme-scan/) | Thematic analysis of recent feedback — cluster signals into actionable themes (coming soon) |
| [`ticket-dig`](./ticket-dig/) | Root cause analysis for a support ticket using session replays and devlogs (coming soon) |
| [`blast-radius`](./blast-radius/) | How many users are affected by a known issue (coming soon) |

### Validate
Build confidence before you commit.

| Skill | What it does |
|---|---|
| [`conviction-doc`](./conviction-doc/) | Draft and stress-test a pre-build conviction doc — evidence, tradeoffs, and kill metrics |
| [`rice-score`](./rice-score/) | RICE scoring grounded in real Pendo data — reach and impact pulled automatically (coming soon) |
| [`idea-rank`](./idea-rank/) | Prioritise roadmap ideas by vote volume and customer segment (coming soon) |
| [`discovery-invite`](./discovery-invite/) | Create a Pendo guide that recruits users for discovery interviews (coming soon) |

### Experiment

| Skill | What it does |
|---|---|
| [`fake-door`](./fake-door/) | Set up a fake door test — guide, click tracking, and results in one command (coming soon) |

### Tag

| Skill | What it does |
|---|---|
| [`tag-new-feature`](./tag-new-feature/) | Tag a new feature in Pendo so everything else can measure it |

### Launch
Ship with confidence, nothing missed.

| Skill | What it does |
|---|---|
| [`launch-checklist`](./launch-checklist/) | End-to-end launch: feature flag, guide, sentiment survey, adoption goal, dashboard (coming soon) |

### Measure
Know if it landed.

| Skill | What it does |
|---|---|
| [`launch-watch`](./launch-watch/) | Weekly launch health update — adoption curve, guide engagement, NPS movement |
| [`bug-watch`](./bug-watch/) | Daily post-launch issue monitor — errors, regressions, support ticket spikes (coming soon) |
| [`exec-brief`](./exec-brief/) | Exec-ready summary of a launch or feature — one paragraph, the numbers that matter (coming soon) |

---

## How it works

Each skill is a folder with a `SKILL.md` file. Claude reads it as a slash command and calls the Pendo MCP for live data. No hardcoded queries, no stale exports — everything runs against your subscription in real time.

---

## Contributing

Built a skill that belongs here? Open a PR. The bar is: does it make product builders faster?

---

MIT License · [pendo-io](https://github.com/pendo-io)
