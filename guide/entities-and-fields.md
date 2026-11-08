# Configuring Entity Types and Fields

Entity types and fields define the schema for your master data. This page explains how to create and configure them. Only **admins** can create or change entity types and fields.

> [!TIP]
> Everyone in the workspace can browse the current schema — every entity type and field they have access to, with descriptions, types and constraints — under **Data Dictionary** in the sidebar. See [The Data Dictionary](data-dictionary.md).

## Creating an entity type

Navigate to **Schemas** under **Configure** in the sidebar and click **+ New Entity Type**. You will need to provide:

- **Display Name** — a human-readable name shown throughout the UI (e.g. "Company").
- **Slug** — a lowercase identifier used in API URLs and the [OData feed](odata.md) (e.g. `company`). Auto-generated from the display name, or set your own. Must start with a letter, contain only lowercase letters, digits, and underscores, and be at most 100 characters. The names `id`, `created_at`, `updated_at`, `entities` and `sources` are reserved. Cannot be changed after creation.
- **Description** — optional notes about this entity type.
- **Auto-merge** — when checked (the default), the golden record is recalculated automatically from the survivorship rules whenever source data changes. Clear it if you want to trigger merges yourself. See [Merging Data Sources](merging.md#triggering-a-merge).
- **Sensitivity** — the baseline [sensitivity label](sensitivity.md) for this entity type. Defaults to "Internal".
- **Retention period (days)** — optional. Data whose last edit is older than this many days is purged automatically. Leave blank to keep data indefinitely. Individual fields can set their own, shorter retention period. See [Retention policies](#retention-policies).
- **Min. Match** — the minimum match score for duplicate and reference candidates shown in data stewardship. Defaults to 50%. Lower values catch more potential duplicates but may include false positives. See [Match threshold](stewardship.md#match-threshold).

Sensitivity labels and retention policies depend on your plan; if they are not included, the settings are shown but disabled. Your plan may also limit how many entity types you can create.

> **Screenshot:** The New Entity Type form, filled in for a "Company" entity type with the auto-generated slug visible.
> <!-- screenshot: images/entities-and-fields-new-entity-type.png -->

## The schema page

After creating an entity type, you land on its **schema page**. This shows all fields in a table, along with their API names, types, sensitivity labels, required and unique settings, and survivorship rules. From here, admins can add, edit, reorder, and remove fields, and edit the entity type itself. Click **View Records** to open the [data grid](viewing-and-editing.md).

Admins can drag fields by the handle at the start of each row to reorder them. The field order determines the column order in the data grid and the API response.

If any field has [source key overrides](#source-key-overrides), a **Key Constraints by Source** table below the fields summarises which fields are unique and required for each source.

> **Screenshot:** The schema page for an entity type, showing the fields table with drag handles, type badges, a sparkle icon on an embedding-enabled field, and the Edit/Remove buttons.
> <!-- screenshot: images/entities-and-fields-schema-page.png -->

## Adding a field

Click **+ Add Field** on the schema page. The **Identity** settings are shown directly on the form; the remaining settings are grouped into sections that you open with their **Configure** button.

> [!TIP]
> Start with at least one unique field. Without a unique key, imports and API upserts can't match incoming data to existing records, and values from different sources can't be merged.

> **Screenshot:** The New Field form, showing the Identity settings and the collapsed Source Key Overrides, Format, Data Processing, Matching and Data Governance sections with their Configure buttons.
> <!-- screenshot: images/entities-and-fields-new-field.png -->

### Identity

- **Display Name** — the column header shown in the data grid and forms.
- **Description** — optional; what the field represents and how it should be used. Shown on the schema page and in the Data Dictionary.
- **API name** — the key used in the API and as the column name in the [OData feed](odata.md). Auto-generated from the display name, or set your own. Same rules as the entity type slug (lowercase letters, digits, underscores, starting with a letter, at most 100 characters); `id`, `created_at`, `updated_at`, `source` and `source_id` are reserved, and the name must be unique within the entity type. Cannot be changed after creation. Amount and quantity fields also get a currency or unit column in the OData feed (`price` → `price_currency`, `weight` → `weight_unit`), so those names can't be used for other fields in the same entity type.
- **Field Type** — determines what kind of data the field holds. See [Field types](#field-types) below. Cannot be changed after creation.
- **Required** — when checked, a value must be provided for this field when creating or updating a record. This is the default for all sources; it can be overridden per source (see below).
- **Unique** — marks the field as part of the entity type's unique key. Records with the same key value are treated as the same record: [imports](importing-data.md) and [API upserts](api.md) update the existing record instead of creating a new one, and values from different sources are matched and merged together. If you mark several fields as unique, together they form a composite key. Marking a field as unique automatically makes it required. Amount and quantity fields cannot be unique.

### Source Key Overrides

If you have [data sources](sources.md) besides manual entry, you can override the required and unique settings for individual sources. This is useful when different source systems provide different subsets of fields — for example, an ERP might require a field that your CRM doesn't have. Add a source from the dropdown, then set its **Required** and **Unique** checkboxes.

Each override can also enable **Defer Errors**, which lets invalid data from that source through without breaking the ingest process; the errors are then flagged for review — see [Deferred Validation Errors](validation-and-lookups.md#deferred-errors).

This section is not available for amount and quantity fields.

### Format

- **Default Value** — pre-filled when creating a new record through the web UI form. Not available for amount and quantity fields.
- **Validation Rules** — constraints that depend on the field type, such as minimum/maximum length or a regex pattern for text, minimum/maximum values and decimal places for numbers, the earliest/latest date for dates, or the list of allowed values for enum fields. See [Validation Rules](validation-and-lookups.md#validation-rules).
- **Convert incoming values to** (quantity fields only) — automatically convert all incoming values to the selected unit.

For [reference fields](#reference-fields), this section is where you choose the referenced entity type.

This section is not shown for lookup and custom field types, whose validation is predefined.

### Data Processing

- **Survivorship Rule** — determines how values from multiple sources are resolved when [merging](merging.md). Defaults to **Source Priority**. **Average** is only available for number, percent and date fields. See [Survivorship Rules](merging.md#survivorship-rules).
- **Transformations** — processing steps applied to incoming values before validation. Not available for amount and quantity fields. See [Transformations](validation-and-lookups.md#transformations).

### Matching

This section is only shown for **text** fields.

- **Enable vector embeddings for matching** — values are converted into vector embeddings in the background and used for duplicate detection, similarity search and [soft reference](#soft-references) matching. Enable this for free-text fields like names and addresses; leave it off for codes, keys, and other structured identifiers. If you enable it on an existing field, embeddings for the field's existing values are generated in the background. Fields with vector embeddings enabled are shown with a sparkle icon on the schema page.
- **Soft reference** — links this text field to a text field on another entity type for cross-entity fuzzy matching. When you choose a target, you also choose the **Target source for matched references**. See [Soft references](#soft-references) below.

### Data Governance

- **Sensitivity** — the field's [sensitivity label](sensitivity.md). The stricter of the field and entity type sensitivity applies. Defaults to the entity type's sensitivity level.
- **Retention period (days)** — values of this field whose last edit is older than this many days are purged automatically. Leave blank to use the entity type's retention period. See [Retention policies](#retention-policies) below.

As for entity types, these two settings depend on your plan.

## Field types

The **Field Type** dropdown groups the available types as follows.

### Built-in types

| Type | Description |
| --- | --- |
| `text` | Free-form text string. Supports min/max length and a regex pattern. |
| `number` | Numeric value. Supports min, max and decimal places. |
| `percent` | Percentage value, entered as e.g. `12.5` or `12.5%` and displayed with a % suffix. Supports the same constraints as number. |
| `date` | Calendar date in `YYYY-MM-DD` format. Supports an earliest and latest date. |
| `boolean` | True/false checkbox. |
| `enum` | One of a fixed set of allowed values, listed in the field's Format settings. |
| `reference` | A link to a record in another entity type. Stored as the referenced record's ID. See [Reference fields](#reference-fields). |
| `amount` | A currency value, stored as a pair of a three-letter currency code and a numeric amount (e.g. `USD 1,234.56`). Supports min, max and decimal places. Cannot be marked as unique. |

### Quantity types

Quantity fields store a numeric value with a unit of measurement (e.g. `100 km`, `3.5 kg`). Each quantity type covers one kind of measurement — such as distance, mass, area, volume, temperature, time, energy or power — and its SI unit is shown in the dropdown. Quantity fields support automatic unit conversion: choose a target unit in the Format settings and all incoming values are converted to it. Quantity fields cannot be marked as unique.

### Lookup types

Lookup fields validate values against a predefined list maintained by Grein, such as country codes and currency codes. Lookup fields also support **alias mapping** — so "United States", "US", and "USA" can all resolve to the same canonical value.

### Custom types

Custom field types are predefined, platform-wide types that enforce a specific format, such as a phone number in international format or an industry classification code. They appear in the field type dropdown under the "Custom" group, and hovering over one shows its description.

## Reference fields

A `reference` field links each record to a record in another entity type — for example, an "Invoice" can reference the "Supplier" it was issued by.

In the field's **Format** settings, choose the **Referenced Entity Type**, then choose a **Lookup field**: one of the unique fields of the referenced entity type. Users and integrations can then enter the lookup value (for example a supplier number) instead of an internal record ID, and Grein resolves it to the matching record. The referenced entity type therefore needs at least one unique field.

## Soft references

A **soft reference** is a cross-entity matching mechanism that automatically suggests which record in another entity type a text value refers to. Unlike a regular `reference` field (which stores an explicit record ID), a soft reference works by comparing text values — through vector similarity when embeddings are enabled, or through text similarity when they are not.

To set up a soft reference:

1. You need a **text field** on your entity type that contains the value to match (e.g. a "Supplier Name" field on an invoice entity type).
2. You need a **text field on another entity type** that contains the values to match against (e.g. the "Name" field on a supplier entity type).
3. You need a **[reference field](#reference-fields)** on your entity type that points to the target entity type (e.g. a "Supplier" reference field). This is where matched references are written.

On the text field's edit page, open **Matching** and set the **Soft reference** dropdown to the target field on the other entity type. Also select a **Target source for matched references** — the source under which matched reference values will be written.

Once configured, match candidates are computed automatically in the background, comparing your values against the golden records of the target entity type. Exact text matches are written immediately as reference values. Near matches above the entity type's **Min. Match** threshold are stored as candidates for manual review on the [reference candidates](stewardship.md#reference-candidates) page in the data stewardship workflow. Records whose reference is already set are not matched again.

Enabling vector embeddings on **both** the source and target text fields improves matching quality, because embeddings capture what a value means rather than just how it is spelled. Without embeddings on both fields, a plain text similarity comparison is used, which handles small spelling differences (e.g. "Jonh Smith" matching "John Smith").

> **Screenshot:** The Matching dialog of a text field, with vector embeddings enabled, a soft reference target selected, and the target source dropdown.
> <!-- screenshot: images/entities-and-fields-soft-reference.png -->

## Editing a field

Click **Edit** next to any field on the schema page. You can change everything except the API name and the field type, which cannot be changed after creation.

## Removing a field

Click **Remove** next to a field on the schema page. Existing data is preserved, but the field no longer appears in the UI, the API, the [OData feed](odata.md) or [natural language queries](ai-query.md). Fields that are used by an [API lookup](validation-and-lookups.md#api-lookups) cannot be removed until the lookup is deleted first.

## Retention policies

Retention policies help you comply with data privacy regulations such as GDPR by automatically removing data that is no longer needed. Retention is measured from the **last edit** of each value — actively maintained data is never purged prematurely.

Retention can be configured at two levels:

- **Entity type level** — applies to every field of the entity type. Each value (in any source) that hasn't been edited within the specified number of days is removed. A record whose values have all been removed is deleted.
- **Field level** — applies to a single field. That field's values are removed in all sources when they haven't been edited within the specified number of days, while the rest of the record is kept. This is useful for sensitive personal data (e.g. date of birth, phone number) that must be purged sooner than the rest of the record.

If both the field and its entity type have a retention period, the **shorter** of the two applies to that field. If neither has one, data is kept indefinitely.

When values are removed, their change history is removed too, and references pointing to a record that has been deleted are cleared.

Retention policies are applied automatically once per day (UTC). See [Audit — Retention](audit.md#retention) for details on how retention interacts with audit data.

## Deactivating an entity type

On the entity type's edit page (click **Edit** on the schema page), admins can **Deactivate** it. Deactivated entity types become read-only — records can still be viewed but not created, updated, imported or deleted, and schema changes are disabled. A deactivated entity type is marked as inactive in the Schemas list and no longer appears in the Data section of the sidebar. It can be re-activated at any time with **Re-activate** on the same page.

To permanently delete an entity type, deactivate it first, then click **Delete** on its edit page. This cannot be undone.

---

[← Back to overview](../README.md)
