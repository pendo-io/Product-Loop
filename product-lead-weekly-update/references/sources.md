# Reading sources

Load this when unsure how to read a source. Patterns are listed by source,
not by tool. Tool and parameter names below are those of the common Claude
connectors; if yours differ, check the loaded tool definitions and use the
equivalent. Every source is optional: if a connector isn't available, skip
that source, say so in the draft handoff, and use whatever the user pastes.

## Meeting notes

Use the meeting-notes connector's query tool with a date filter for the
window (for Notion: `notion-query-meeting-notes` with a `created_time`
filter, relative value `the_past_week`, or an exact date range). It usually
defaults to notes where the current user is an attendee or creator, so no
person filter is needed. The query returns a list of notes (title,
attendees, date, URL), not their content. Fetch each note by ID with the
connector's fetch tool (for Notion, `notion-fetch`) to get its summary.
Prefer the summary block; read the transcript only when the summary is
missing, since some fetch tools return the full transcript by default.
Never paste a transcript into the rollup. Cap each meeting at roughly
2,000 tokens of context; summarize longer ones before drafting.

## Feature tracker

Used to derive the team (product areas → features → owners) and to check a
PM's claim against a feature record's status. Search tools often can't
filter by a person or relation field; use the tracker's query tool instead
(for Notion, `notion-query-data-sources`). Fetch a feature's record only
when a PM update or other source points at it. See `tracker-mode.md` for
the derivation query and Notion filter tips.

## Slack: read a channel in a window

```
slack_read_channel
  channel_id: "<channel id>"
  oldest: "<window start as a Slack ts string: Unix seconds with a decimal part>"
  latest: "<window end as a Slack ts string>"
  limit: 100
```

Most connectors cap `limit` at 100. Messages come back newest first. For
high-traffic channels, search instead of reading everything (see below).
Don't list every channel in a large workspace; start from the configured
channels.

## Slack: read a DM

Most Slack connectors accept a user ID as `channel_id` for DM history, so
no separate conversation ID is needed:

```
slack_read_channel
  channel_id: "<user id>"
  oldest: "<window start>"
  latest: "<window end>"
  limit: 100
```

## Slack: read one thread

```
slack_read_thread
  channel_id: "<channel id>"
  message_ts: "<parent message ts>"
  oldest: "<window start>"
  latest: "<window end>"
```

## Slack: search for mentions of a feature

```
slack_search_public_and_private
  query: "\"<feature name>\""
  after: "<window start Unix timestamp>"
  before: "<window end Unix timestamp>"
  limit: 20
```

Use the `after` / `before` parameters rather than dates in the query. The
query supports modifiers such as `in:channel-name`, `from:@user`,
`is:thread`, `has:link`. Run searches only for the features being covered,
and only after the configured channel reads.

## Slack: resolve a handle (setup only)

`slack_search_users` with a name, handle, or email. Take the top match only
if it's an exact email or full-name match; otherwise confirm with the user.

## Slack: send a DM

`slack_send_message` with the user ID as `channel_id` and the text in
`message`. Use `slack_send_message_draft` instead only if the user wants to
edit the message before it goes out.

## Google Drive

- Read a file: `read_file_content` with `fileId` (camelCase).
- Check before reading: `get_file_metadata` with `fileId` and
  `excludeContentSnippets: true`; if `modifiedTime` is before the window,
  skip the content fetch.
- Scan a folder for recent edits: `search_files` with a query such as
  `parentId = '<folder id>' and modifiedTime > '<window start, RFC 3339 UTC>'`.
  The syntax is `parentId = 'X'`, not `'X' in parents`. Use RFC 3339 UTC
  timestamps.

## Confluence

- Read a page: `getConfluencePage` with your site's `cloudId` (the site
  URL usually works too), `pageId`, and
  `contentFormat: "markdown"`.
- Recent edits in a space: `searchConfluenceUsingCql` with
  `space = '<spaceKey>' and lastmodified > '<YYYY-MM-DD>'`, `limit: 25`.
  Fetch full content only for pages whose titles look relevant.
- If a call fails: retry with the site URL as `cloudId`; then try the
  equivalent tool in another loaded Atlassian namespace; then use the
  connector's search and fetch by the returned ID; then skip the source and
  note it in the draft handoff.

## Output size discipline

- Cap each PM update at about 3,000 tokens of body content; summarize
  longer ones internally before synthesizing.
- Cap meeting context at about 2,000 tokens per meeting.
- Read at most 100 Slack messages per channel call (`limit: 100`).
- Don't fetch every feature linked from every PM update; fetch only the
  ones whose status the rollup will explicitly cover.
- Don't fetch every milestone or every comment; fetch only what changed in
  the window.
- For Confluence and Jira, search first, then fetch the few relevant items.
