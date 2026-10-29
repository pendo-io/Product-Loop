# Product Loop

**Your product is already telling you what to build next.**

The teams that win aren't the ones who hear it first — they're the ones who act on it in the same breath. Product Loop puts a Claude skill at every turn of that loop: discover what's worth solving, validate before committing, launch with confidence, measure what landed.

Signal to decision to impact. Hours, not weeks.

Every skill is backed by live data from your Pendo subscription.

Works with Claude Code, the Claude apps, and Cowork.

**Community skills, provided as is.** These skills are shared by Pendo’s product team under the MIT License. They are not a supported Pendo product, and there is no SLA or warranty. You are responsible for the data you connect and only use sources you are authorized to use.

---

## Install

Product Loop is a plugin marketplace. Install everything as the `product-loop` plugin, or one stage as a bundle (see [Bundles](#bundles)).

**Claude Code** (terminal, desktop app, IDE):

```
/plugin marketplace add pendo-io/Product-Loop
/plugin install product-loop@product-loop
```

Swap `product-loop@product-loop` for a bundle, for example `product-loop-validate@product-loop`. In the desktop app you can also use + > Plugins > Add plugin. Or run `./setup [bundle]` from a clone of this repo, which does the same through the `claude` CLI. Skills load as `/product-loop:<skill>`, for example `/product-loop:pulse`.

**Claude.ai web and Desktop chat:** Customize > Plugins > Add marketplace, then enter `pendo-io/Product-Loop`. Skills need code execution and file creation turned on.

**Cowork:** Customize > Plugins > Add marketplace, then enter the GitHub URL of this repo.

**One skill only:**
- Claude Code: copy the skill's folder to `~/.claude/skills/<skill-name>/`, so its `SKILL.md` sits directly inside it.
- Claude apps: download this repo as a ZIP, re-zip just that skill's folder (with `SKILL.md` at the top of the folder), and upload it under Customize > Skills.

Then authenticate Pendo once:

```
/mcp
```

The plugin's `.claude-plugin/plugin.json` points the Pendo MCP server at the US host. If your Pendo subscription is in another region, change that URL to your region's host.

---

## Bundles

Each bundle installs one stage's skills. `product-loop` installs all of them. Bundles can overlap, and a skill installed by two bundles shows up once.

| Bundle | Skills |
|---|---|
| `product-loop` | Every skill below |
| `product-loop-monitor` | pulse, weekly-prio, feedback-idea-producer, field-qual-producer, support-trends-producer, engineering-progress-producer, competitor-release-producer, attention-radar, daily-digest-conductor, feature-validation, product-manager-weekly-update, product-lead-weekly-update |
| `product-loop-uncover` | funnel-dropoff-investigator, friction-finder |
| `product-loop-validate` | conviction-doc, gtm-scoring, prototype-fidelity |
| `product-loop-launch` | feature-creation, release-note-drafter, product-announcement-generator, release-kit |
| `product-loop-measure` | launch-watch |

---

## Skills

### Monitor
Keep a constant pulse on what's happening.

| Skill | What it does |
|---|---|
| [`pulse`](./pulse/) | Weekly product health digest — usage trends, feedback themes, guide performance |
| [`weekly-prio`](./weekly-prio/) | Pull, cluster, and prioritise recent feedback from Pendo Listen — link top themes to backlog ideas |
| [`feedback-idea-producer`](./feedback-idea-producer/) | Daily or weekly digest of what customers are saying: feedback themes typed as bug, request, pain point, or praise, linked to backlog ideas |
| [`field-qual-producer`](./field-qual-producer/) | What customers and prospects say about your areas on calls you weren't on, led by themes that recur across accounts |
| [`support-trends-producer`](./support-trends-producer/) | Support-ticket themes spiking or fading in your areas, plus tickets waiting on engineering with no linked bug |
| [`engineering-progress-producer`](./engineering-progress-producer/) | What shipped, what's blocked, and what newly broke in your issue tracker (Jira, GitHub Issues, Linear, and others) since the last run |
| [`competitor-release-producer`](./competitor-release-producer/) | Delta-only read of competitor releases that touch your product areas |
| [`attention-radar`](./attention-radar/) | Folds the producer skills' output into one short daily digest, one linked sentence per item |
| [`daily-digest-conductor`](./daily-digest-conductor/) | Runs your daily skills on their own cadence and posts one digest: what it handled, what needs your OK, what needs you |
| [`feature-validation`](./feature-validation/) | Checks your feature records are still accurate and brings back only the ones that need a human |
| [`product-manager-weekly-update`](./product-manager-weekly-update/) | Drafts a PM's weekly status update from the week's meeting notes, Slack, docs, and owned features |
| [`product-lead-weekly-update`](./product-lead-weekly-update/) | Rolls a team's PM updates into one leadership update and flags where they disagree with other sources |
| `bug-radar` | Daily scan for emerging bugs and errors before your users report them (coming soon) |
| `usage-watch` | Daily usage checker — flag drops, spikes, and anomalies in key features (coming soon) |

### Uncover
Find the real problems worth solving.

| Skill | What it does |
|---|---|
| [`funnel-dropoff-investigator`](./funnel-dropoff-investigator/) | Diagnose where users abandon a flow — funnel metrics, session replays, and qual feedback in one report |
| [`friction-finder`](./friction-finder/) | Blends rage clicks, drop-off, and qual feedback to surface where users are stuck |
| `theme-scan` | Thematic analysis of recent feedback — cluster signals into actionable themes (coming soon) |
| `ticket-dig` | Root cause analysis for a support ticket using session replays and devlogs (coming soon) |
| `blast-radius` | How many users are affected by a known issue (coming soon) |

### Validate
Build confidence before you commit.

| Skill | What it does |
|---|---|
| [`conviction-doc`](./conviction-doc/) | Draft and stress-test a pre-build conviction doc — evidence, tradeoffs, kill metrics, and a GTM launch score |
| [`gtm-scoring`](./gtm-scoring/) | Score a launch across four dimensions (experience, adoption, competitive, enablement) and map it to a GTM tier. The rubric is a basic example to adapt to your own launch process |
| [`prototype-fidelity`](./prototype-fidelity/) | Build a UI prototype from a screenshot, treating the screenshot as the spec, with an optional usability audit |
| `rice-score` | RICE scoring grounded in real Pendo data — reach and impact pulled automatically (coming soon) |
| `idea-rank` | Prioritise roadmap ideas by vote volume and customer segment (coming soon) |
| `discovery-invite` | Create a Pendo guide that recruits users for discovery interviews (coming soon) |

### Experiment

| Skill | What it does |
|---|---|
| `fake-door` | Set up a fake door test — guide, click tracking, and results in one command (coming soon) |

### Tag

| Skill | What it does |
|---|---|
| `tag-new-feature` | Tag a new feature in Pendo so everything else can measure it (coming soon) |

### Launch
Ship with confidence, nothing missed.

| Skill | What it does |
|---|---|
| [`feature-creation`](./feature-creation/) | Turn a conviction doc or feature idea into a feature record by type (Net-new, Enhancement, or Light), with a GTM score and roadmap description where they apply |
| [`release-note-drafter`](./release-note-drafter/) | Draft the customer-facing release-note entry for a shipped or shipping change |
| [`product-announcement-generator`](./product-announcement-generator/) | Draft a short internal launch announcement: why care, who's affected, what's next |
| [`release-kit`](./release-kit/) | Write customer-facing launch content: blog, slides, social, FAQs, email, landing page, and in-app guide copy |
| `launch-checklist` | End-to-end launch: feature flag, guide, sentiment survey, adoption goal, dashboard (coming soon) |

### Measure
Know if it landed.

| Skill | What it does |
|---|---|
| [`launch-watch`](./launch-watch/) | Weekly launch health check: adoption curve, guide engagement, funnel, friction, and sentiment, as a shareable dashboard where the surface supports one, otherwise an HTML file or text block |
| `bug-watch` | Daily post-launch issue monitor — errors, regressions, support ticket spikes (coming soon) |
| `exec-brief` | Exec-ready summary of a launch or feature — one paragraph, the numbers that matter (coming soon) |

---

## How it works

Each skill is a folder with a `SKILL.md` file. Claude reads it as a slash command and calls the Pendo MCP for live data. No hardcoded queries, no stale exports — everything runs against your subscription in real time.

---

## Contributing

Built a skill that belongs here? Open a PR. The bar is: does it make product builders faster?

When you add a skill, also add its folder to `.claude-plugin/marketplace.json`: to the `product-loop` plugin's `skills` list, and to the stage bundle it belongs in. A skill missing from that file isn't installed by the plugin.

---

[MIT License](./LICENSE) · [pendo-io](https://github.com/pendo-io)
