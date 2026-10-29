# Setup

Run this when the `product_lead_weekly` config section doesn't exist and the
user wants to set up, or when they ask to redo part of their setup. One
focused question per step. Use a multiple-choice tool for closed choices if
the client has one, prose for free text. Save incrementally so an
interrupted setup resumes from the first missing field. Every step is
skippable; a skipped step falls back to its default or to "ask at run time".

## Initial check

Explain what the skill does in two sentences at most, then ask:

> Set up your weekly rollup now? It takes about 3 minutes.
> 1. Yes, set up now
> 2. Later, just draft this week's rollup

If "Later", go to the weekly run and don't write config, apart from the
one output question the run asks (SKILL.md → Output).

## Step 1: Identify the user

Ask for (or infer from connected tools) the lead's name, timezone, and, if
connected, their identity in the tracker and Slack. If a
`pm_weekly` section exists in the shared config, offer to reuse its
identity fields. If the user looks like a PM rather than a lead (for
example they own features but no product areas), mention that
`product-manager-weekly-update` may be what they want, and continue if they confirm.

```json
"user": { "display_name": "Sam Okafor", "timezone": "America/Denver", "tracker_user_id": null, "slack_user_id": null }
```

## Step 2: Audience

Who reads the rollup? A name or role, for example "VP of Product". Used in
phrasing only.

```json
"audience": { "name": "VP of Product" }
```

## Step 3: Product areas

The areas this lead owns. They define the team (Step 4) and, in tracker
mode, the value of the rollup's product-area relation.

- **Tracker connected:** find areas whose lead or owner field points at
  this user (read the schema first to find that field; in some trackers it
  links to a people record rather than the user directly). Show the list:
  > I found <N> product areas owned by you. Use them all?
  > 1. Use all of them
  > 2. Let me prune the list
  > 3. Add more / start over
- **No tracker:** ask for the area names.

```json
"team": { "product_areas": [ { "name": "Workload View", "url": null }, { "name": "Timesheets", "url": null } ] }
```

## Step 4: PMs on the team

- **Tracker connected:** derive PMs from ownership: features in the lead's
  areas → each feature's owner. Union and dedupe. **Don't filter by a role
  or title field**; those are often unset and silently drop most of the
  team. The lead may appear if they own a feature; that's fine. Show the
  list:
  > I found <N> PMs in your product areas. Use them all as your team?
  > 1. Use all
  > 2. Let me prune
  > 3. Add more
  Then: re-derive the team each run, or freeze this list? Default:
  re-derive (handles people joining or leaving mid-quarter).
- **No tracker:** ask for the PMs' names, one per line.

```json
"team": { "auto_detect": false, "pms": [ { "display_name": "Jordan Rivera", "tracker_user_id": null } ] }
```

## Step 5: Where PM updates come from

> Where should I find your PMs' weekly updates?
> 1. I'll paste them each week
> 2. A folder (for example where product-manager-weekly-update saves them)
> 3. A tracker database they write to

- **Folder:** ask for the path. If a `pm_weekly` section exists with an
  `output.markdown_folder`, offer that.
- **Tracker:** ask for the database URL; validate per
  `tracker-mode.md`. If a `pm_weekly` section exists with a tracker
  target, offer that.

```json
"pm_updates": { "source": "paste", "folder": null, "tracker_url": null }
```

## Step 6: Slack sources (optional, for the cross-check)

Leads usually read higher-level channels than PMs (a leadership channel,
key cross-team channels) plus a few DMs.

1. Ask for channels (names or IDs, comma- or newline-separated). Resolve
   names with the Slack connector's channel search; confirm ambiguous
   matches.
2. Ask for DMs (handles, one per line). Resolve each to a user ID; most
   Slack connectors accept a user ID as the channel for DM reads and sends.
3. Ask which channels, if any, to read in full: "Any small channels where
   anyone's post could matter (for example an engineering or incident
   channel)? I'll read every post there instead of filtering to posts by
   you, mentioning you, or naming an area or PM." Set `read_all: true` on
   those entries; the default is `false`. Personal, HR, and bot posts are
   still left out of those channels (see Gotchas in `SKILL.md`).

```json
"sources": { "slack": [ { "type": "channel", "handle": "#team-channel", "id": "<channel id>", "read_all": false } ] }
```

## Step 7: Doc sources (optional)

Same as `product-manager-weekly-update`: Notion, Google Drive, or Confluence URLs, one
per line. Validate each by fetching it.

```json
"sources": { "docs": [ { "type": "google_drive_file", "url": "<url>", "name": "Area goals" } ] }
```

## Step 8: Output

> Where should the rollup go?
> 1. Markdown in chat (default)
> 2. Markdown in chat, and save a copy to a folder
> 3. My tracker (a Notion database or similar)

For a tracker, ask for the destination URL and validate per
`tracker-mode.md` (title property, optional plain-text summary property,
optional product-area relation).

```json
"output": { "mode": "markdown", "markdown_folder": null, "tracker": { "target_url": null, "title_property": null, "summary_property": null, "product_areas_property": null } }
```

## Step 9: Template

> Rollup format?
> 1. Default template (headline / updates / customer commitments / risks / coming up)
> 2. Match an existing rollup I'll link or paste
> 3. Custom, I'll describe it

Then ask once:

> Does your team track customer commitments (promises to named customers, with a status)?
> 1. Yes, include them in the rollup
> 2. No (default)

If no, the rollup skips the customer commitments prompt.

```json
"template": { "source": "default", "reference": null, "custom_notes": null, "tracks_customer_commitments": false }
```

## Step 10: Schedule (optional)

> Run this automatically each week?
> 1. Yes, on your team's rollup day (tell me which day)
> 2. No, I'll ask when I want it

If the lead doesn't name a day, use the first working day of their week
and say it's the default. Under the
`daily-digest` conductor, skip this step: the conductor stores the day as
its `cadence` for this skill. Then time (default: early on that day,
before your team's meetings) and timezone if unknown. Show the scheduling
steps in SKILL.md → "Scheduling (optional)".

```json
"schedule": { "enabled": false, "day": "<your team's rollup day>", "time": "<HH:MM>", "timezone": "America/Denver" }
```

## Step 11: Tone and absent PMs

> How should I write the rollup?
> 1. Match my prior rollups (recommended)
> 2. Structured: bolded initiative header (no emoji unless your prior rollups use them) + sub-bullets in a fixed layout
> 3. Prose: paragraphs grouped by initiative
> 4. Hybrid

> When a PM hasn't filed by run time, what do you want?
> 1. Mention them by name with "no update"
> 2. Omit them
> 3. Add an "Awaiting updates from: X, Y" footer (default)

```json
"style": { "mode": "match_prior", "fallback_style": "structured", "absent_pm_handling": "footer" }
```

## Step 12: Review step (tracker mode only)

> Want a heads-up when your rollup is created?
> 1. Yes, tell me in chat
> 2. Yes, tell me in chat and send me a Slack DM
> 3. No, just publish

Show option 2 only when Slack is connected. Save exactly what the user
chose:

| Choice | Saved |
| --- | --- |
| 1. Chat | `{ "enabled": true, "notify": "chat", "always_notify_on_discrepancy": true }` |
| 2. Chat and Slack DM | `{ "enabled": true, "notify": "slack_dm", "always_notify_on_discrepancy": true }` |
| 3. No heads-up | `{ "enabled": false, "notify": "chat", "always_notify_on_discrepancy": true }` |

Option 3 keeps `notify: "chat"` so a discrepancy notice still has a
channel. For option 2, the DM goes to `user.slack_user_id` (Step 1). If
that is empty, resolve the lead's handle now (`references/sources.md` →
Slack: resolve a handle) and save it.

```json
"review_step": { "enabled": true, "notify": "slack_dm", "always_notify_on_discrepancy": true }
```

`notify` is `chat`, `slack_dm`, or `none`. `slack_dm` also tells the user
in chat.

`always_notify_on_discrepancy` means: even if `enabled` is false, notify
when the run flagged discrepancies. Leads generally want to know.

## Wrap up

1. Add `"version": "1.0.0"`, `created_at`, `updated_at`, `"last_run": null`.
2. Write the section to the config file named in SKILL.md → Config (`config.json`) under
   `product_lead_weekly` (create the file if absent; leave other sections
   alone). Read it back to confirm it parses.
3. If scheduling was chosen, show the scheduling checklist.
4. Confirm in one short paragraph where settings are saved, when it runs,
   and how to change part of it ("add a PM to my team", "change my
   schedule").
5. If the user originally asked for this week's rollup, go straight to the
   weekly run.

## Partial re-setup

- "update my product areas" → Step 3 (then Step 4 if `team.auto_detect`)
- "add a PM to my team" / "redo my PMs" → Step 4
- "change where PM updates come from" → Step 5
- "update my Slack sources" → Step 6
- "change my schedule" → Step 10
- "change my review setup" → Step 12
- "redo everything" → clear the section and start at Step 1

Always update `updated_at`. Don't touch other fields or sections.
