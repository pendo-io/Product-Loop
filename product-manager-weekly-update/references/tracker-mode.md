# Tracker mode (bring your own tracker)

Load this only when `config.output.mode` is `tracker` or the user asks to
write the update to a tracker. Markdown in chat is the default and needs
none of this.

## Rules

1. **Read before you write.** Fetch the destination's schema first. Map the
   update's parts to existing properties by name:
   - the title → the title property
   - the features covered → a relation or multi-select to the features
     tracker, if one exists
   - the body → the page body
2. **Never create properties or options.** If something has no matching
   property, keep it in the body and say so in chat.
3. **Never write auto-set or computed fields**: created time, created by,
   rollups, formulas.
4. **One record per update.** Create it in one call (properties and body
   together); don't split property and content writes.
5. **Link only what the body covers.** The feature relation mirrors the
   body, never the full owned-features list.
6. **Return the record's URL** to the user.

## Setup validation

When the user gives a destination URL during setup:

1. Fetch it and confirm it's a database or project you can create records in.
2. Find the title property and a property that links to features. Record
   their exact names in `config.output.tracker`.
3. If there's no feature link property, tell the user the update will list
   features in the body only.
4. If the database holds several data sources (Notion multi-source
   databases), store the data source ID, not the database ID, so the create
   call targets the right one.

## Notion examples

The tips below are Notion examples. They apply to Notion's MCP tools and
save a lot of silent failures; other trackers need their own equivalents.

- **Create a page:** `notion-create-pages` with
  `parent: { data_source_id: "<id>" }`. Property values must be strings,
  numbers, or null, so a relation is passed as a JSON-stringified array of
  page URLs, for example
  `"Features": "[\"https://www.notion.so/<page1>\", \"https://www.notion.so/<page2>\"]"`.
  Don't repeat the title at the top of `content`.
- **Title with mentions:**
  `<mention-user url="user://<user id>"/> - PM update - <mention-date start="YYYY-MM-DD"/>`.
  Inline mentions in the body use the same syntax
  (`<mention-page url="..."/>`, `<mention-date start="..."/>`).
- **Finding a user's prior updates by author fails silently if the form is
  wrong.** In SQL, `Created by` matches only as `notion_user-<uuid>` or with
  `LIKE '%<uuid>%'`; `user://<uuid>` and the bare uuid return zero rows that
  look exactly like "no prior updates". Prefer rows mode, which takes the
  natural value:

  ```json
  {
    "mode": "rows",
    "data_source_url": "<updates data source URL>",
    "limit": 3,
    "filter": {
      "type": "group", "operator": "and",
      "filters": [{
        "type": "property", "property": "Created by", "propertyType": "created_by",
        "operator": "person_contains",
        "value": { "type": "exact", "value": "user://<user id>" }
      }]
    },
    "sort": [{ "property": "<created time property>", "direction": "descending" }]
  }
  ```

  Don't match on the title either: plain-text titles carry no user ID.
- **Filtering a person or relation field in SQL:** use `"<column>" LIKE ?`
  with the wildcards in the parameter (`"%<id>%"`). Relation and person
  cells are JSON arrays of URLs or `user://` IDs, so a substring match
  works.
- **Search tools can't filter by relation.** Use `notion-query-data-sources`
  for "features owned by X".
- **No drafts.** A Notion page exists or it doesn't, so a review step means
  creating the page and asking the user to edit it in place. Don't create
  it somewhere else and move it; the URL would change and break relations.
