# The Data Dictionary

The data dictionary is a read-only reference of every entity type in your workspace and the fields it contains. It brings descriptions, field types, sensitivity labels and key constraints together on a single page, so that anyone working with your data, whether in the web UI, through the [REST API](api.md) or in a reporting tool connected to the [OData feed](odata.md), can see what each field means and how it is named.

## Opening the data dictionary

Click **Data Dictionary** under **Tools** in the sidebar. The page is available to every member of the workspace, including viewers. You don't need a particular [role](users-and-permissions.md#roles) to open it.

The dictionary is built from your current schema each time you open the page, so it always reflects the latest changes to entity types and fields.

> **Screenshot:** The data dictionary page showing two entity types, each with its description and a table of fields.
> <!-- screenshot: images/data-dictionary-overview.png -->

## What the dictionary shows

Each entity type gets its own section, headed by its display name followed by its machine name. Click the display name to open the entity type's [schema page](entities-and-fields.md#the-schema-page). If the entity type has a description, it is shown under the heading.

Below that, a table lists the entity type's fields:

| Column | Description |
|---|---|
| **Field** | The field's display name. |
| **API name** | The field's machine name, as used in the [REST API](api.md) and the [OData feed](odata.md). |
| **Type** | The [field type](entities-and-fields.md#field-types), for example `text`, `number` or `reference`. |
| **Description** | The field's description, if one has been entered. |
| **Sensitivity** | The field's *effective* [sensitivity level](sensitivity.md). See [Sensitivity in the dictionary](#sensitivity-in-the-dictionary) below. |
| **Required** | "Yes" if a value must be provided for the field. |
| **Unique** | "Yes" if no two records may share the same value for the field. |

> [!NOTE]
> The **Required** and **Unique** columns show the default constraints for each field. Overrides configured for individual sources are not shown here. You can see them on the entity type's [schema page](entities-and-fields.md#the-schema-page).

Entity types that have been [deactivated](entities-and-fields.md#deactivating-an-entity-type), and fields that have been [removed](entities-and-fields.md#removing-a-field), are left out of the dictionary. If an entity type has no fields yet, its table says "No fields defined", and if the workspace has no entity types at all, the page says "No entity types defined yet".

> [!TIP]
> The dictionary is only as useful as the descriptions in it. Admins can add or update descriptions for entity types and fields from the [schema page](entities-and-fields.md#editing-a-field).

## Sensitivity in the dictionary

The data dictionary uses the same [sensitivity rules](sensitivity.md#how-visibility-is-determined) as the rest of the platform, so different users can see different versions of the page:

- **Entity types** whose sensitivity level is higher than your own access level are left out entirely.
- **Fields** whose effective sensitivity is higher than your access level are left out of their entity type's table.

A field's effective sensitivity is the stricter of the field's own label and its entity type's label. The **Sensitivity** column always shows this effective level. If the level comes from the entity type rather than the field's own label, the badge carries the same [overridden indicator](sensitivity.md#the-overridden-indicator) used on the schema page.

> [!NOTE]
> Hidden entity types and fields are left out without any notice, so you can't tell from the dictionary whether anything has been hidden from you. If you expect to see something that isn't listed, ask a workspace admin to check your [sensitivity level](users-and-permissions.md#changing-a-users-sensitivity-level).

## Exporting and printing

The data dictionary has no export or download option of its own. To get a copy, use your browser's print function, or retrieve the schema programmatically through the [REST API's schema endpoints](api.md#schema-endpoints) or the [OData feed's](odata.md) metadata. Keep in mind that a printout shows only the entity types and fields visible at your own sensitivity level.

---

[← Back to overview](../README.md)
