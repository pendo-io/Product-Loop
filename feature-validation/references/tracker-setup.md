# Tracker setup (bring your own tracker)

Load in tracker mode: when features come from a tracker, or when the PM opted into write-back. Covers schema introspection, finding due features, and write payloads. Nothing here applies to the pasted-list, markdown-only path.

## Rules

- **Read the schema first.** Fetch the database / project / base schema before any query or write, and resolve the actual name, type, and options of every property in `config.fields`. Store them in `config.resolved_property_names`.
- **Never create properties or options.** If a property is missing, drop it from this run (for a high-signal field) or fall back to markdown for that write (for `Last Validated` / `Validated by`), and tell the PM what to add if they want it.
- **Use exact option labels and casing.** A mismatch can silently create a duplicate option in some trackers.
- **Never write computed fields** (formulas, rollups, lookups).
- **Ownership is enforced in the query and re-checked before every write.**

## Properties the skill uses

Default names; map each to the tracker's actual name in `config.fields`.

| Role (`config.fields` key) | Default name | Typical type | Needed for |
| --- | --- | --- | --- |
| `title` | `Feature` | title | naming features in the report |
| `owner` | `Product Manager` | person / relation / user | own-features-only scope |
| `current_phase` | `Current phase` | select | shipped-phase exclusion, high-signal field |
| `shipped_phase` | `GA` | option label | which phase is out of scope |
| `due_flag` | `Due for Validation` | formula / checkbox | routine due (optional) |
| `pre_change_flag` | `Pre-change due` | formula / checkbox | pre-change due (optional) |
| `last_validated` | `Last Validated` | date | stamp; also the no-flags due rule |
| `validated_by` | `Validated by` | select, options `Agent`, `User` | stamp |
| `spec_link` | `Spec link` | url | evidence |
| `milestones` | `Milestones` | relation | target-date evidence (optional) |
| `roadmap_description` | `Description for roadmap` | text | read only, to flag empty descriptions for feature-creation's Backfill mode (optional) |
| `show_on_roadmap` | `Show on customer-facing roadmap?` | checkbox or select (Yes/No) | read only, to skip off-roadmap features in the empty-description check; unmapped means nothing is skipped (optional) |
| high-signal fields | see `SKILL.md` | varies | research and Tier C edits |

At setup, read the type of each high-signal field. Common gotchas: a Yes/No field may be a select, not a checkbox (write the option string); regions are often a multi-select; target dates are dates; early access details are text.

## Finding due features

One query, server-side where possible: **owner is the PM** AND **current phase is not the shipped phase** AND (**due flag true** OR **pre-change flag true**). If the tracker has no due flags, apply `config.due_rule.cadence_days` as described in `SKILL.md` step 3.

### Notion

Notion formula properties may be flagged as not SQL-queryable, and a page fetch can return a formula's value as an opaque reference rather than the boolean. The path that works is a **rows-mode query with a formula `resultFilter`**, which resolves the formula server-side:

```json
{
  "mode": "rows",
  "data_source_url": "<your Features data source URL>",
  "limit": 10,
  "filter": {
    "type": "group", "operator": "and", "filters": [
      { "type": "property", "property": "<owner property>", "propertyType": "relation",
        "operator": "relation_contains", "value": { "type": "exact", "value": "<PM's people page URL>" } },
      { "type": "property", "property": "<current phase property>", "propertyType": "select",
        "operator": "enum_is_not", "value": { "type": "exact", "value": "<shipped phase>" } },
      { "type": "group", "operator": "or", "filters": [
        { "type": "property", "property": "<due flag>", "propertyType": "formula", "operator": "any",
          "resultFilter": { "type": "property", "property": "<due flag>", "propertyType": "checkbox",
            "operator": "checkbox_is", "value": { "type": "exact", "value": true } } },
        { "type": "property", "property": "<pre-change flag>", "propertyType": "formula", "operator": "any",
          "resultFilter": { "type": "property", "property": "<pre-change flag>", "propertyType": "checkbox",
            "operator": "checkbox_is", "value": { "type": "exact", "value": true } } }
      ] }
    ]
  }
}
```

If the owner property is a person field rather than a relation, use the person filter with the PM's user instead. If it's a relation to a people database, resolve the PM's page in that database first: search or query the people database by the PM's name (`pm_user.display_name`), confirm the match with the PM if more than one page comes back or the name isn't exact, and use that page's URL in the filter (save it as `pm_user.tracker_user` so later runs skip the lookup). The rows come back with the high-signal field values inline, so a per-feature fetch is only needed for the body, notes, and related milestones.

**Keep query limits small.** Rows mode returns every property of every row, including long text fields, so even a few rows make a large payload. Use a small `limit` (about 10) and page through the results if the query tool supports it, rather than one large request. Once the due set is known, fetch only the fields you need for each feature.

### Jira

Use JQL, for example `assignee = currentUser() AND status != "<shipped status>"`, or filter on whichever field holds the feature owner, plus the due flags if they exist as fields.

### Airtable and others

Filter by the owner field and shipped phase, and on the due flags if the base computes them (Airtable formula fields can be filtered with `filterByFormula`).

## Reading context for research

To corroborate a target date that rolls up from related milestones, read the milestones' own target dates and status. Don't write the rollup.

## Writing a validation

**Precondition on every write:** the invoking user is the feature's owner, and the feature is not in the shipped phase. Otherwise write nothing and skip it.

One update per validated feature, for example in Notion:

```
properties: {
  "date:<Last Validated>:start": "<today, YYYY-MM-DD>",
  "date:<Last Validated>:is_datetime": 0,
  "<Validated by>": "Agent"          // "User" for Tier B/C
}
```

For **Tier C**, set the PM-agreed corrected fields in the same update, using each field's write form (select = option string; multi-select = the tracker's list form; date = date; text = string; empty clears). Never edit a high-signal field in Tier A or B.

Then add the audit comment (format in `validation-logic.md`). If the tracker has no comments, write the audit line to a notes field the PM named at setup, or print it in chat.
