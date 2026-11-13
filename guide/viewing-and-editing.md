# Viewing and Editing Data

The data grid is the main interface for browsing, editing and managing records.

## Opening the data grid

Click an entity type's name under **Data** in the sidebar to open its data grid. Admins can also click an entity type's name on the **Schemas** page, or **View Records** on its schema page.

The grid shows records as rows and fields as columns. The column order matches the field order defined on the [schema page](entities-and-fields.md). Unique fields are highlighted, and required fields are marked with an asterisk (\*). Records are sorted by their unique field values.

> **Screenshot:** The data grid with the search box, source dropdown, Merge, Export and Schema buttons, and rows with green and amber left borders.
> <!-- screenshot: images/viewing-and-editing-grid.png -->

## Switching sources

The **source dropdown** in the header switches between the data from different [sources](sources.md). Select **Merged results** to see the golden records. Select **Manual entry** (the default) to see values entered through the web UI. Other sources show data contributed by imports and external systems.

The grid only lists records that have at least one value in the selected source.

Editing is only available when viewing the **Manual entry** source.

## Searching

Type a search term into the search box and press **Enter**. The grid filters to records that contain the term in any text value — in any field and from any source, not just the one you are viewing. Clear the search by clicking the **×** button next to the search box.

## Advanced filters

Click the filter icon in the search box to open the advanced filter panel. Click **+ Add filter** to add a condition: pick a field, an operator (`=`, `≠`, `>`, `<`, `≥`, `≤`, **contains**, **is empty** or **is not empty**) and a value. Add as many conditions as you need — a record must match all of them — and click **Apply**. Filters apply to the values in the selected source. Click **Clear all** to remove them.

## Inline editing

If you have editor or admin access and are viewing the Manual entry source, you can click any cell in the grid to edit it. You can also move between cells with the arrow keys and start editing by pressing Enter or F2, or simply by typing. Press Enter or Tab (or click elsewhere) to finish editing a cell, or Escape to cancel. To clear a cell, select it and press Delete (or Backspace). Changes are saved automatically a moment after you finish editing.

If a value fails validation, the cell is outlined in red — hover over it to see the error message — and the row is not saved until the problem is fixed.

The grid has an editor for each field type: text and number inputs, dates typed as `YYYY-MM-DD`, checkboxes for booleans, dropdowns for enum fields, reference fields and custom list types, text inputs with suggestions for lookup types (such as countries and currencies), amount fields (currency + value) and quantity fields (value + unit).

### Default currency and units

An admin can choose a default currency, and a default unit for each kind of quantity (for example `ft` for distance or `lb` for mass), under **Settings → Defaults**. When you start editing an empty amount or quantity cell, the default currency or unit is already filled in. The same applies to the **New record** form. For an empty amount cell, the cursor starts in the currency with the default selected: type a code to replace it, or press Tab to move to the amount. If you start typing a digit instead, it goes straight into the amount.

The default is only a suggestion. You can choose another currency or unit, and nothing is saved until you enter a number. Existing values, imports and the API are not affected. The defaults apply to the whole workspace, and the currencies and units to choose from are the same ones that amount and quantity fields accept.

> **Screenshot:** The Defaults dialog in Settings, with a currency selected and a default unit chosen for a few quantity types.

Edits through the grid are saved to the Manual entry source. Fields you don't have [sensitivity clearance](sensitivity.md) to see are not shown, and their values are left unchanged.

## Creating a record

There are two ways to create a record:

- **Inline:** when viewing the Manual entry source with edit access, an empty row marked with `*` is always present at the bottom of the grid. Fill in the fields, and the record is created as soon as the row is saved.
- **Form:** add `/new` to the end of the grid's address to open a full-page form. It shows all fields with labels, validation hints and error messages.

If the unique field values you enter match exactly one existing record — for example, one that was created by an import — your values are added to that record under Manual entry instead of creating a new one.

## Deleting a record

With edit access on the Manual entry source, each row has a delete button (**×**) in the last column. Clicking it asks for confirmation.

If other records reference this one (through [reference fields](entities-and-fields.md#field-types)), a dialog lists them and offers three choices:

- **Cancel — do not delete anything.**
- **Clear references, then delete this record** — the referencing records are kept, but their reference field is emptied.
- **Delete all referencing records, then delete this record.**

Deleting from the grid removes the record's Manual entry values. If no other source has values for the record, the record itself is deleted. If other sources still have values for it, the record remains, with their data.

## Record history

Click a record's **ID number** (the `#` column) to view its history. The history page shows:

- The current **merged** values (the golden record).
- The changes made by each source, newest first, showing which fields changed, when and by whom.

If the record has unmerged changes (new source data that has not been merged into the golden record yet), a notice is shown at the top.

In the data grid, records that have been merged are marked with a green left border. Records with unmerged changes have an amber left border. See [Merging Data Sources](merging.md) for how merging works.

> **Screenshot:** A record's history page with the merged values at the top and the change history grouped by source.
> <!-- screenshot: images/viewing-and-editing-history.png -->

## Exporting data

Click the **Export** button in the data grid header to download the data as a CSV file. The export uses the current source selection, search and advanced filters, but includes all matching records rather than just the current page (up to 100,000 records).

The CSV uses the field display names as column headers. Values are formatted for readability: dates as `YYYY-MM-DD`, booleans as `true`/`false`, amounts as `USD 1234.56`, and quantities as `100 km`. Reference fields contain the ID number of the referenced record.

Only fields you have [sensitivity clearance](sensitivity.md) to see are included in the export. All roles, including viewers, can export.

## Pagination

The grid shows 50 records per page. Use the pagination controls below the grid to move between pages. The total number of records is shown above the grid.

## Deactivated entity types

If an entity type has been deactivated, its grid shows a notice and all records are read-only.

## Sensitivity filtering

The data grid only shows fields that are within your [sensitivity clearance](sensitivity.md). If your clearance level is lower than the entity type's sensitivity, you cannot open its data grid at all. See [Working with Sensitivity Labels](sensitivity.md) for details.

---

[← Back to overview](../README.md)
