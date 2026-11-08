# Merging Data Sources

When multiple [sources](sources.md) contribute values for the same record, the platform consolidates them into a single golden record. This process is called **merging**, and it is controlled by **survivorship rules** configured on each field. The golden record is shown as the **Merged results** source.

## Survivorship rules

Each field has a survivorship rule that determines how conflicting values from different sources are resolved. You choose it under **Data Processing** when adding or editing a field. The available rules are:

### Source Priority

The default rule. The value from the highest-priority source that has a value for the field wins. Source priority is determined by the order on the [Sources page](sources.md#source-priority) — sources higher in the list have higher priority.

### Most Recent

The most recently updated value wins, regardless of which source it came from. If two values were updated at the same time, source priority breaks the tie. This is useful for fields where the latest information is always the most accurate.

### Consensus

The value that the most sources agree on wins. If there is a tie, source priority decides among the tied values.

### Average

For number, percent and date fields only. The merged value is the arithmetic mean of all source values. This rule cannot be selected for other field types.

### Median

The merged value is the middle value when all source values are sorted (with an even number of values, the higher of the two middle values). Unlike Average, this works with any field type and is more robust against outliers.

## What is merged

- Only **complete** records are merged — records that have a value for every unique key field. Incomplete records are left out until they are resolved through [data stewardship](stewardship.md#what-makes-a-record-incomplete). Entity types without unique fields are merged regardless.
- Values that have [deferred validation errors](validation-and-lookups.md#deferred-errors) are ignored by the survivorship rules until they are fixed.
- When a record is deleted, its merged values are removed.

## Triggering a merge

### Automatically

Each entity type has an **Auto-merge** setting on its create and edit pages. It is enabled by default. With auto-merge on, a background process brings the golden records up to date shortly after source data changes — whether through the web UI, the API, an import or a stewardship action. You do not need to do anything.

### Manually

If auto-merge is turned off, or you want to bring the golden records up to date immediately, you can trigger a merge yourself:

- **From the web UI:** open the [data grid](viewing-and-editing.md) for an entity type. Admins see a **Merge** button in the header. Click it and confirm.
- **From the API:** send a `POST` request to `/api/v1/{entityTypeName}/merge`. See [Trigger a merge](api.md#trigger-a-merge) in the REST API documentation.

A merge processes every record of the entity type that has unmerged changes.

> **Screenshot:** The data grid header with the Merge button, and rows showing the green (merged) and amber (unmerged changes) left borders.
> <!-- screenshot: images/merging-grid-merge-button.png -->

## What happens during a merge

For each record with unmerged changes:

1. For each field, the values from all sources are collected.
2. The field's survivorship rule picks the winning value.
3. The winning value is written to the **Merged results** source.
4. The record is marked as merged (shown with a green left border in the data grid).

Records that have received new data since the last merge are shown with an amber left border, indicating they have unmerged changes.

If merging changes a record's merged values, the record is checked again for [potential duplicates](stewardship.md#reviewing-potential-duplicates), since duplicate detection compares merged values.

## Merging and record matching

When data arrives through the [API](api.md), an [import](importing-data.md) or manual entry, the platform uses the entity type's **unique fields** to match incoming data to existing records. If a match is found, the incoming values are added to that record under the source they came from. If no match is found, a new record is created.

New data is then merged into the golden record — automatically if auto-merge is on, otherwise the next time a merge is triggered.

## When to merge manually

If auto-merge is turned off for an entity type, consider merging after:

- An API integration pushes a batch of new data.
- You import data from a file or spreadsheet.
- You resolve issues in [data stewardship](stewardship.md).

There is no harm in merging frequently. If nothing has changed since the last merge, the golden records stay the same.

---

[← Back to overview](../README.md)
