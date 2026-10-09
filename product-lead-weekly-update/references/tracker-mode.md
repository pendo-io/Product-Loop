# Tracker mode (bring your own tracker)

Load this when reading PM updates from a tracker, deriving the team from a
tracker, or writing the rollup to a tracker. Markdown in chat with pasted
PM updates needs none of this.

## Rules for writing the rollup

1. **Read before you write.** Fetch the destination's schema first. Map the
   rollup's parts to existing properties by name:
   - the title → the title property
   - the Summary → a plain-text summary property, if one exists
   - the product areas → a relation or multi-select to product areas, if
     one exists; set it to every area in `config.team.product_areas`
   - the body → the page body
2. **Never create properties or options.** If something has no matching
   property, keep it in the body and say so in chat.
3. **Never write auto-set or computed fields**: created time, created by,
   rollups, formulas. For example, if a property is a rollup the tracker
   computes from another relation, leave it out of the payload; a tracker
   may accept the value and silently discard it.
4. **One record per rollup**, created in one call.
5. **Never edit the PM updates.**
6. **Return the record's URL** to the user.

## Setup validation

When the user gives a destination URL:

1. Fetch it; confirm you can create records in it.
2. Record the exact names of the title property, the summary property (if
   any), and the product-area property (if any) in
   `config.output.tracker`.
3. For multi-source databases (Notion), store the data source ID.

## Reading PM updates from a tracker

Query the PM updates database once per PM for records created from the
window start up to the run date (so late filers count), filtered by author (created by, or an author/person property if
the database has one). Fetch each result's body. Treat a PM as absent only
after a query in a form known to work returns nothing; a wrong filter form
often returns zero rows silently, which looks exactly like "didn't file".

## Deriving the team

The join chain is **lead → product areas → features → feature owner**:

1. Find the lead's product areas (the product-area record's lead/owner
   field points at the lead; in some trackers it points at a people record,
   so resolve the lead's people record first).
2. For each area, find features whose area relation includes it.
3. Collect each feature's owner field. Union and dedupe.
4. **Don't filter by a role or title field.** Those are often unset, and
   filtering on them silently drops most of the team.

If a cross-database join isn't supported, do it in two passes: query the
features for the owner values per area, then look up each owner.

## Notion tips

- **Create a page:** `notion-create-pages` with
  `parent: { data_source_id: "<id>" }`. Property values must be strings,
  numbers, or null, so a relation is a JSON-stringified array of page URLs,
  for example
  `"Product Areas": "[\"https://www.notion.so/<area1>\", \"https://www.notion.so/<area2>\"]"`.
  Don't repeat the title in `content`.
- **Title with mentions:**
  `<mention-user url="user://<user id>"/> Update - <mention-date start="YYYY-MM-DD"/>`.
  If the user's ID isn't known, use the plain-text title.
- **Summary as a plain-text property:** join the bullets with `<br>`, not
  real newlines, and use no markdown and no mentions (they don't render in
  a text property).
- **Author filters fail silently if the form is wrong.** In SQL, `Created
  by` matches only as `notion_user-<uuid>` or with `LIKE '%<uuid>%'`;
  `user://<uuid>` and the bare uuid return zero rows. Prefer rows mode with
  `person_contains` and the natural `user://<uuid>` value:

  ```json
  {
    "mode": "rows",
    "data_source_url": "<PM updates data source URL>",
    "filter": {
      "type": "group", "operator": "and",
      "filters": [{
        "type": "property", "property": "Created by", "propertyType": "created_by",
        "operator": "person_contains",
        "value": { "type": "exact", "value": "user://<pm user id>" }
      }]
    },
    "sort": [{ "property": "<created time property>", "direction": "descending" }]
  }
  ```

  Then keep the rows created from the window start up to the run date. In SQL form:

  ```sql
  SELECT url, "Name", "Created by", "<created time property>", "<features property>"
  FROM "<PM updates data source URL>"
  WHERE "Created by" = ? AND "<created time property>" >= ? AND "<created time property>" < ?
  ```

  with `params: ["notion_user-<pm user id>", "<window start ISO>", "<run date ISO, end of day>"]`.
- **Filtering a person or relation field in SQL:** `"<column>" LIKE ?` with
  the wildcards in the parameter (`"%<id or url>%"`). Relation cells are
  JSON arrays of page URLs.
- **No drafts.** Review means creating the page and editing it in place.
