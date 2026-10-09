# Sources: the extensible source model

The conductor lets a user watch **as many sources as they want**, of any supported type, without the config schema ever changing. Two pieces make this work:

1. **`config.sources`**: an open-ended *list* of typed source objects (not a fixed set of keys).
2. **The source-type registry** (below): declares each supported `type`, what connector it needs, how to read it, and which members consume it by default.

Adding one more Slack channel, or a whole new *kind* of source, is appending to the list and, for a brand-new kind, adding one row to the registry.

## Source object shape

```json
{
  "id": "src-004",                 // stable, unique; never reused
  "type": "slack_channel",         // must exist in the registry below
  "label": "#workload-team",       // human-friendly name for digests/questions
  "ref": "<channel id>",           // the identifier the connector needs (see registry)
  "keywords": ["workload"],        // optional: filter reads to matching content
  "consumers": "all",              // "all" | ["product-manager-weekly-update", "digest", ...]
  "enabled": true                  // toggle without deleting
}
```

- **`id`** is stable so a source can be edited or removed by id and referenced in `last_run`.
- **`consumers`** maps the source to the members that should read it. `"all"` = every member that can consume this `type`. A list scopes it (for example a noisy channel feeding only the digest, not the weekly update). Default when omitted: `"all"`.
- **`keywords`** are optional read filters, useful for busy channels.
- **`enabled: false`** parks a source without losing its config.

There is **no upper limit** on sources. Past ~25 sources for one member in a run, warn (don't block): that many slows the run and adds noise.

## Source-type registry

| type | connector | `ref` is | how it's read | default consumers |
|---|---|---|---|---|
| `slack_channel` | Slack | channel id | read the channel over the window; filter by `keywords` / owned-feature names | all |
| `slack_dm` | Slack | user id | read the DM over the window | weekly writers |
| `tracker_page` | Notion, Confluence, or similar | page URL/id | fetch; note edits/comments in the window | all |
| `tracker_database` | Notion, Airtable, or similar | database URL/id | query rows changed in the window | all |
| `meeting_notes` | tracker or docs tool | `auto` or a location | query notes in the window | weekly writers |
| `gdrive_doc` | Google Drive | doc URL/id | read; note recent edits | weekly writers |
| `gdrive_folder` | Google Drive | folder URL/id | list and read changed files in the window | weekly writers |
| `jira_filter` | Jira | JQL / filter id | search issues changed in the window | feature-validation, weekly writers |
| `url_rss` | web fetch | URL | fetch and diff since last run | digest |
| `call_recordings` | a call recorder (for example Gong) | library / filter / team ref | read recent call transcripts or notes in the window | field-qual-producer |
| `pendo_usage` | Pendo MCP | subscription id + entity ids (see `pendo` config) | usage reads for adoption and trends | launch-watch |
| `pendo_listen` | Pendo MCP | subscription id + product-area ids | feedback items and insights | feedback-idea-producer |
| `email_thread` | an email connector | query / label | search relevant threads in the window (read-only) | none yet (no current member reads email; set `consumers` when one does) |

`digest` is a pseudo-consumer meaning "feed the conductor's own Exec roll-up," not a specific member. `attention-radar` is never a source consumer: it reads only the producers' outputs, so a source reaches the digest through the producer that reads it.

### Call notes: a fallback chain, not a single source

"Call notes" (customer-call signal, read by `field-qual-producer`) resolve through a **fallback chain**, using the first that's connected:

1. **A call recorder** (`call_recordings`): primary. If the connector fails, **prompt the user to re-authorize it** before giving up.
2. **Meeting notes** (`meeting_notes`) in the tracker or docs tool, if the recorder can't be reached.
3. **Any other notes tool the user names**, last, and only after asking.

Store the working choice in `config.radar.call_notes_source` so the chain doesn't re-prompt every run, and pass that source to `field-qual-producer`, which picks its own retrieval mode from it; re-check only on failure.

**The registry is the single source of truth.** If a `type` in `config.sources` isn't in this table, flag it (🔴 "unknown source type X; add it to the registry") and skip it rather than guessing.

## Deriving member inputs from the list

Members expect their own shapes (for example a weekly writer wants `slack_sources[]` and `doc_sources[]`). The conductor **derives** those at run time by filtering the generic list; it doesn't store them twice:

- `slack_sources` for a member = sources whose `type` is in {`slack_channel`, `slack_dm`} and whose `consumers` include the member.
- `doc_sources` for a member = sources whose `type` is in {`tracker_page`, `tracker_database`, `meeting_notes`, `gdrive_*`} and whose `consumers` include the member.
- …and so on per member.

When a member has its own config, pass these derived arrays as shared inputs; the member fills gaps from its own config.

## Adding, editing, removing sources

Triggered by "add a source", "watch this channel too", "stop watching X", or onboarding Step 3. For each add:

1. Ask for the source (free text: a channel name, a page URL, and so on).
2. **Infer the `type`** from the input: `slack_channel` for a `#channel`, `tracker_page` for a Notion URL, `gdrive_doc` for a Drive URL. Confirm if ambiguous.
3. **Resolve the `ref`** with the registry's connector (turn a channel name into an id; keep the URL for docs).
4. Optionally ask for `keywords` and `consumers` (default `"all"`), then append with a new `id` and `enabled: true`.
5. **Dedup:** if the same `type` + `ref` exists, update it in place.

Removing = `enabled: false` (soft) or drop by `id` (hard) on request.

## Connector gating

At setup and run start, compute the connector union from enabled sources plus enabled members. If a source needs a connector that isn't connected, mark it `blocked`, skip it, and surface one 🔴 line ("Slack not connected; 3 sources skipped"). Never fail the whole run for one unreachable source.

## Record vs. source of reality

A tracker is where current reality is **recorded**; it is rarely where that reality is **created**. Customer calls, product usage, feedback, and live team discussion show change first; the roadmap record lags. attention-radar's fixed category order reflects this (customer signal first, the record last), and when a producer's signal disagrees with the record, the digest surfaces the record as possibly stale rather than trusting it by default. A single uncorroborated signal never overrides a settled record. Feature validation and the weekly writers still treat the tracker as their target of truth.

## New source types arriving via an update

When a newer member or conductor version introduces a **new source type**, don't silently enable it or silently ignore it. Prompt: *"New source available: {type}; want to connect it?"* If yes, run the add flow and set its `consumers`; if no, record the decline so it isn't re-asked. **Nothing is hardwired; every source is opt-in.**
