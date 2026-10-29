# Output channels: where the digest and notifications go

The daily digest (and any other conductor notification) is delivered through a **pluggable output channel**, chosen the same way sources are: a typed entry backed by a registry. The default is the chat the run happens in; moving the digest to a Slack DM, or adding a tracker page or a team channel, is a config change, never a rewrite. Fan-out to several channels at once is supported.

## Config shape

```json
"digest": {
  "channels": [
    { "type": "chat", "ref": null, "enabled": true }
  ],
  "quiet_days": false
}
```

- **`channels`** is a *list*: deliver to one or many. Each has `type` (from the registry below), `ref` (the target the connector needs, for example a Slack user or channel id; `null` where not needed), and `enabled`.
- The **bucket structure is channel-agnostic**: ✅ / 🟡 / 🔴 render into whatever each channel supports.
- `quiet_days` applies across all channels.

## Output-channel registry

Add a row to support a new destination. The digest composition doesn't change; only the delivery adapter does.

| type | connector | delivers via | `ref` is | notes |
|---|---|---|---|---|
| `chat` | none | a message in the run's own chat (the scheduled task's chat, or this conversation) | `null` | default; always available |
| `slack_dm` | Slack | a DM to the user | user id | |
| `slack_channel` | Slack | a message to a channel | channel id | team or roll-up digests (for example a PL sharing with their group) |
| `tracker_page` | Notion or similar | append or update a dated digest block on a page | page id/URL | persistent, searchable record |
| `email` | an email connector | a message to the user | address | only when an email connector with send is connected |

If a `type` isn't in the registry, fall back to `chat` and note it once. If a channel's connector isn't connected, skip that channel (deliver to the others) and add a 🔴 line the next time `chat` is available.

## Choosing channels

During setup (or "change where my digest goes"), default to `chat`. Offer the others as a multi-select: "Deliver your daily digest to: this chat / Slack DM / a Slack channel / a tracker page (pick any)." Resolve each `ref` with the right connector. Because it's a list, a user can DM themselves *and* keep a tracker record.

## Affordances by surface (what actually renders)

The digest editor (`references/digest-editor.md`) relies only on affordances that are real for the target:

- **chat**: hyperlinks ✓; interactive questions ✓ where the client has a question tool; no native expand (use links or "ask me to show the rest").
- **slack_dm / slack_channel**: hyperlinks ✓; no interactive questions and no expand; put detail in a thread; decisions are resolved when the user opens the run in chat.
- **tracker_page**: hyperlinks ✓; toggle blocks for collapse/expand where the tracker has them; no inline questions.

When a digest fans out to several channels, the content is the same but these affordances degrade per channel; the interactive question flow always happens in chat.
