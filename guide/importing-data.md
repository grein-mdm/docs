# Importing Data

The import feature lets you paste delimited data (CSV, TSV or similar) into the browser, or load it from a file, and map it to your entity type's fields. This is the fastest way to load a batch of records without using the [API](api.md).

## Starting an import

Click **Import data** in the sidebar. You need editor or admin access.

Choose an **Entity type** and a **Destination**. The entity type dropdown only shows active entity types that you have [sensitivity clearance](sensitivity.md) to access. The destination determines which [source](sources.md) the imported values are attributed to; you can choose any source except Merged results.

> **Screenshot:** The import page with the Entity type and Destination dropdowns and the paste area.
> <!-- screenshot: images/importing-data-start.png -->

## Pasting data or loading a file

Paste your data into the text area, drag and drop a file onto it, or click **browse** to pick a file. Text files (`.csv`, `.tsv`, `.txt`) of up to 50 MB are supported.

The first row must be a header row with column names. Values may be enclosed in double quotes, so that they can contain the delimiter or line breaks. The importer automatically detects the delimiter, trying the following in order:

1. Tab
2. Pipe (`|`)
3. Semicolon (`;`)
4. Comma (`,`)
5. Space

The detected delimiter is shown below the text area. Data with only one column is also accepted.

> [!TIP]
> Copying a range of cells from Excel or Google Sheets and pasting it into the text area works well — spreadsheets use the tab-delimited format when copying to the clipboard.

> [!NOTE]
> Apache Parquet files can't be imported on this page. To import a Parquet file, upload it through the [REST API](api.md).

## Mapping columns

Once data is pasted and an entity type is selected, a **Column mapping** table appears. Each row shows a column header from your data and a dropdown to select the field it maps to. Unique fields are marked with ★.

The importer auto-matches columns by comparing the header text (ignoring case) with the fields' display names and API names. Review the mappings and correct any that were not matched automatically. Leave a column set to **— skip —** to exclude it from the import. Each field can only be mapped to one column.

Some field types have extra options:

- **Amount fields** expect a currency code and a number, for example `GBP 123.45` or `123.45 GBP`.
- **Quantity fields** show a second dropdown where you choose the unit for the column. Values that are just a number are read in that unit; values that include a unit, like `100 km`, keep their own.

Reference fields are not available for mapping, and only fields you have [sensitivity clearance](sensitivity.md) to see are listed.

> **Screenshot:** The column mapping table with auto-matched fields, a skipped column and a unit dropdown for a quantity field, followed by the preview summary.
> <!-- screenshot: images/importing-data-mapping-preview.png -->

## Previewing

Click **Preview** to validate the data without writing anything. Each row is processed the same way an import would process it: [transformations](validation-and-lookups.md#transformations) are applied, and the values are checked against the field's [validation rules](validation-and-lookups.md#validation-rules), using the required and unique settings for the destination source. The preview shows:

- **Rows** — how many data rows were parsed.
- **Valid** — how many rows passed validation.
- **Errors** — rows that failed validation, with row numbers and error messages (the first 20 are listed).
- **Sample rows** — up to 5 valid rows in a table, for visual confirmation.

If you change the mappings, click **Preview** again.

Rows whose only errors are in fields with [Defer Errors](validation-and-lookups.md#deferred-errors) enabled count as valid. They are imported with the error flagged, so that it can be fixed later in [data stewardship](stewardship.md#fixing-deferred-validation-errors).

## Committing the import

Click **Confirm import** to write the data. If some rows have errors, the button reads **Import *n* valid rows (skip *m* errors)**: the valid rows are imported and the rows with errors are skipped.

- Rows that match an existing record (by unique key) **update** that record.
- Rows with no match **create** new records.

After the import, you are taken to the entity type's data grid, showing the destination source.

An import is refused if it could exceed your plan's record limit or storage limit. The pasted data is kept for 30 minutes after the first preview; if you wait longer than that, preview it again before importing.

## Upsert behaviour

The import uses the entity type's **unique fields** to decide whether a row is a new record or an update to an existing one. A row updates an existing record when the row's unique field values all match the record's unique field values. Otherwise, a new record is created. Rows without any unique field values always create new records.

If [Source Key Overrides](entities-and-fields.md#source-key-overrides) are configured for the destination source, its unique and required settings are used instead of the defaults.

When a row updates a record:

- Only the destination source's values are changed; values from other sources are left alone.
- Empty cells and unmapped columns leave the existing value unchanged — an import does not clear values.
- If nothing actually changed, the record is not touched.

The imported values are then merged into the golden record — automatically if [auto-merge](merging.md#triggering-a-merge) is enabled for the entity type, otherwise the next time a merge is run. New and changed records are also checked for [potential duplicates and other data issues](stewardship.md) in the background.

---

[← Back to overview](../README.md)
