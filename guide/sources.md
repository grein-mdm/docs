# Working with Sources

Sources represent where your data comes from. Each source contributes its own values for each record, and Grein merges them into a single golden record based on source priority and [survivorship rules](merging.md#survivorship-rules).

## Built-in sources

Every workspace comes with two built-in sources that cannot be removed:

- **Merged results** (`merged`) — the consolidated golden record, produced by combining values from all other sources. This is a read-only view; you cannot write to it directly. It is always first in the priority order and cannot be moved.
- **Manual entry** (`manual`) — the source used for data entered and edited in the web UI's data grid and record form.

## Creating a source

Navigate to **Sources** under **Configure** in the sidebar (admin only). Under **Add source**, enter a display name (e.g. "ERP System") and a slug (e.g. `erp_system`) for the new source, then click **Add source**. New sources are added at the bottom of the priority order.

The slug identifies the source in the [REST API](api.md) and must start with a letter and contain only lowercase letters, digits, and underscores. Each slug can only be used once — this includes sources that have been removed, so a removed source's slug can't be reused.

> **Screenshot:** The Sources page, showing the priority-ordered list of sources with drag handles and the Add source form below.
> <!-- screenshot: images/sources-sources-page.png -->

## Source priority

Sources are ranked by priority, from the top of the list on the Sources page. When merging values using the **Source Priority** survivorship rule, Grein picks the value from the highest-priority source that has a value for that field.

To change the priority order, drag sources by their handle into the desired position on the Sources page. Merged results always stays at the top.

> [!NOTE]
> Source priority only directly affects the "Source Priority" survivorship rule. Other rules like "Most Recent" or "Consensus" use different criteria to pick the winning value. See [Merging Data Sources](merging.md).

## Per-source required and unique fields

Different source systems often provide different subsets of fields. On each field, you can override whether it is required or unique for a specific source, and allow that source's validation errors to be deferred for review. See [Source Key Overrides](entities-and-fields.md#source-key-overrides).

## Viewing data by source

On the [data grid](viewing-and-editing.md#switching-sources), the source dropdown in the header lets you switch between viewing values from different sources. The default view is "Manual entry". Select "Merged results" to see the golden records. Editing in the grid is only possible when viewing Manual entry.

Each source may have different values for the same record. Comparing sources helps you understand where data discrepancies exist before merging. A record's history page also shows its values grouped by source.

## Writing data to a source

Data lands in a source in several ways:

- **Web UI** — records created or edited in the data grid or the record form are stored in the **Manual entry** source.
- **Import** — when you [import data](importing-data.md), you choose a **Destination** source. Any source except Merged results can be chosen, including Manual entry.
- **REST API** — every API write request must specify a `source`, either in the request body or as a query parameter. The source must exist and cannot be `merged` or `manual`. See [The REST API](api.md).
- **API lookups** — enrichment results are written to the source configured on the lookup. See [API Lookups](validation-and-lookups.md#api-lookups).
- **Soft references** — matched references are written to the target source configured on the soft reference field. See [Soft references](entities-and-fields.md#soft-references).
- **[Attribute extraction](ai-extraction.md)** — AI-extracted values are written to the output source you choose when running the extraction.

## Removing a source

You can remove any source you have created by clicking **Remove** next to it on the Sources page. The source will no longer appear in the UI or be available for new writes, but existing data is preserved.

> [!WARNING]
> A source cannot be removed if it is used by an [API lookup](validation-and-lookups.md#api-lookups). Delete the lookup first.

---

[← Back to overview](../README.md)
