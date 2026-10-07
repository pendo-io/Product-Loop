# Setup

Run this when the `pm_weekly` config section doesn't exist and the user
wants to set up, or when they ask to redo part of their setup. Each step is
one focused question. Use a multiple-choice tool for closed choices if the
client has one, prose for free text. Save incrementally, so an interrupted
setup resumes from the first missing field.

Every step is skippable. A step the user skips falls back to its default or
to "ask at run time".

## Initial check

Explain what the skill does in two sentences at most, then ask:

> Set up your weekly update now? It takes about 3 minutes.
> 1. Yes, set up now
> 2. Later, just draft this week's update

If "Later", go to the weekly run and don't write config, apart from the
one output question the run asks (SKILL.md → Output).

## Step 1: Identify the user

Ask for (or infer from connected tools) the PM's name, timezone, and, if
connected, their identity in the tracker and Slack (used for the tracker
title mention, self-mention searches, and review DMs). If a lookup returns
several matches, confirm with the user.

```json
"user": {
  "display_name": "Jordan Rivera",
  "timezone": "America/Denver",
  "tracker_user_id": null,
  "slack_user_id": null
}
```

## Step 2: Audience

Who reads this update? Store a name or role (for example "my manager" or
"Head of Product"). It is used in phrasing ("what should I tell <audience>")
and has no effect on the format.

```json
"audience": { "name": "my manager" }
```

## Step 3: Meetings

If a meeting-notes connector is connected (Notion meeting notes, Granola,
or similar), there's nothing to set up: notes for the window are pulled at
run time for meetings the user attended or created. If none is connected,
the user can paste notes at run time. If the user wants meetings excluded,
record it in `style.notes`.

## Step 4: Slack sources (optional)

1. Ask: *"Which Slack channels should I watch for your update? Paste names
   (`#team-channel`) or IDs, one per line or comma-separated."*
2. Resolve each name with the Slack connector's channel search; confirm
   ambiguous matches.
3. Ask: *"Which DMs should I watch? List Slack handles, one per line."*
   Resolve each to a user ID. Most Slack connectors accept a user ID as the
   channel for DM reads and sends.

```json
"sources": {
  "slack": [
    { "type": "channel", "handle": "#team-channel", "id": "<channel id>" },
    { "type": "dm", "handle": "@manager.handle", "user_id": "<user id>" }
  ]
}
```

## Step 5: Doc sources (optional)

Ask: *"Which docs should I watch for changes? Paste URLs: Notion pages or
databases, Google Drive files or folders, Confluence pages or spaces. One
per line."* Validate each by fetching it; if a fetch fails, say which one
and ask the user to re-paste or skip.

```json
"sources": {
  "docs": [
    { "type": "confluence_page", "url": "<url>", "name": "Design decisions" },
    { "type": "google_drive_file", "url": "<url>", "name": "Beta feedback tracker" }
  ]
}
```

## Step 6: Features

The skill needs to know which features the PM owns.

- **Tracker connected:** ask for the features database or project URL, read
  its schema, and ask which field holds the owner (for example `Product
  Manager`). Query features where that field contains the user and show the
  list:
  > I found <N> features assigned to you. Use them all?
  > 1. Use all of them
  > 2. Let me prune the list
  > 3. None of these are right
  If "prune", let them deselect (batch the choices if the tool caps the
  number of options, or ask for a comma-separated list to drop). If "none",
  ask them to paste feature names or URLs.
  Then ask: auto-detect owned features each run, or freeze the list?
  Default: auto-detect.
- **No tracker:** ask for a list of feature names (and links, if any), or
  "whatever I paste each week".

```json
"features": {
  "source": "tracker",
  "tracker_url": "<features database URL>",
  "owner_field": "Product Manager",
  "auto_detect": true,
  "list": []
}
```

## Step 7: Output

> Where should the update go?
> 1. Markdown in chat (default)
> 2. Markdown in chat, and save a copy to a folder
> 3. My tracker (a Notion database or similar)

- **Folder:** ask for the path (default `./weekly-updates/`). The
  product-lead skill can read PM updates from a shared folder.
- **Tracker:** ask for the destination database URL and follow the
  validation steps in `references/tracker-mode.md`.

```json
"output": {
  "mode": "markdown",
  "markdown_folder": null,
  "tracker": { "target_url": null, "data_source_id": null, "title_property": null, "features_property": null }
}
```

## Step 8: Template

> Update format?
> 1. Default template
> 2. Match an existing update I'll link or paste
> 3. Custom, I'll describe it

- **Default** → the four prompts in `references/update-format.md`.
- **Match existing** → fetch or read it, extract the structural pattern
  (grouped vs. flat), bullet vs. paragraph style, and mention conventions.
- **Custom** → store the description verbatim and treat it as an override.

```json
"template": { "source": "default", "reference": null, "custom_notes": null }
```

## Step 9: Schedule (optional)

> Run this automatically each week?
> 1. Yes, on your team's update day (tell me which day)
> 2. No, I'll ask when I want it

If the user doesn't name a day, use the last working day of their week
and say it's the default. Under the
daily-digest-conductor, skip this step: the conductor stores the day as
its `cadence` for this skill. Then time (default: late in that day, with
time left to review) and timezone if unknown. Then give the user the
scheduling steps in SKILL.md → "Scheduling (optional)". The skill can't
register a scheduled task by itself; the user confirms it in their client.

```json
"schedule": { "enabled": true, "day": "<your team's update day>", "time": "<HH:MM>", "timezone": "America/Denver" }
```

## Step 10: Tone and length

> Tone and length?
> 1. Match my prior updates (recommended)
> 2. Short bullets only
> 3. Narrative paragraphs
> 4. Hybrid

"Match my prior updates" uses the PM's last 2 or 3 updates as style anchors
(voice and body structure) at run time. If the user asks what Hybrid
means, use the one definition in `references/update-format.md` → Voice.

```json
"style": { "mode": "match_prior", "fallback_style": "hybrid", "notes": null }
```

## Step 11: Review step (tracker mode only)

Trackers like Notion have no draft state, so "review" means the record is
created and the user edits it in place. Ask whether they want a heads-up:

> Want a heads-up when your update is created, so you can review it?
> 1. Yes, tell me in chat
> 2. Yes, tell me in chat and send me a Slack DM
> 3. No, just publish

Show option 2 only when Slack is connected. Save exactly what the user
chose:

| Choice | Saved |
| --- | --- |
| 1. Chat | `{ "enabled": true, "notify": "chat" }` |
| 2. Chat and Slack DM | `{ "enabled": true, "notify": "slack_dm" }` |
| 3. No heads-up | `{ "enabled": false, "notify": "none" }` |

For option 2, the DM goes to `user.slack_user_id` (Step 1). If that is
empty, resolve the user's handle now (`references/sources.md` → Slack:
resolve a handle) and save it.

```json
"review_step": { "enabled": true, "notify": "slack_dm" }
```

`notify` is `chat`, `slack_dm`, or `none`. `slack_dm` also tells the user
in chat.

## Wrap up

1. Add `"version": "1.0.0"`, `created_at`, `updated_at`, and `"last_run":
   null`.
2. Write the section to the config file named in SKILL.md → Config (`config.json`) under `pm_weekly`
   (create the file if absent; leave other sections alone). Read it back to
   confirm it parses.
3. If scheduling was chosen, show the scheduling checklist.
4. Confirm in one short paragraph where the settings are saved, when it
   runs (if scheduled), and how to change part of it ("update my Slack
   sources", "change my schedule").
5. If the user originally asked for this week's update, go straight to the
   weekly run.

## Partial re-setup

- "update my Slack sources" → Step 4
- "add a feature I own" / "redo my features" → Step 6
- "change where it goes" → Step 7
- "change my schedule" → Step 9
- "change my review setup" → Step 11
- "redo everything" → clear the `pm_weekly` section and start at Step 1

Always update `updated_at`. Don't touch unrelated fields or other skills'
sections.
