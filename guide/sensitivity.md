# Working with Sensitivity Labels

Sensitivity labels control who can see what data. They provide a simple, hierarchical classification system that applies across the web UI, the [REST API](api.md), the [OData feed](odata.md) and [natural language query](ai-query.md).

## The four levels

Grein uses four sensitivity levels, from least to most restricted:

1. **Public** — unrestricted data that can be shared freely.
2. **Internal** — the default level. Suitable for general business data that should not be publicly exposed.
3. **Sensitive** — confidential data that should only be accessible to a limited group.
4. **Secret** — the most restricted level, for highly confidential data.

## Where labels are assigned

Sensitivity labels are set in four places:

- **Entity types** — sets a baseline for the entire entity type. Configured with the **Sensitivity** setting when you create or edit the entity type.
- **Fields** — individual fields can have their own sensitivity level. Configured on the field's edit page, in the **Data Governance** section.
- **Users** — each user has an access level (clearance) per workspace. Configured in the **Access Level** column of the user list under [Settings](users-and-permissions.md#changing-a-users-sensitivity-level).
- **API keys** — each key has its own sensitivity level, chosen when [creating the key](api-keys.md#creating-an-api-key).

New entity types, users and API keys default to **Internal**. A new field defaults to the sensitivity of its entity type.

Labelling entity types and fields is not included in every plan. Without it, the **Sensitivity** settings on entity types and fields are disabled with the hint "Upgrade your plan to configure sensitivity labels": new entity types get Internal, new fields get their entity type's level, and existing labels are kept. User access levels and API key sensitivity can always be set.

> **Screenshot:** The Data Governance dialog of a field, with the Sensitivity dropdown and the hint showing the entity type's sensitivity.
> <!-- screenshot: images/sensitivity-field-governance.png -->

## How visibility is determined

A user (or API key) can see a field only if their access level is **equal to or higher than** the field's effective sensitivity.

The **effective sensitivity** of a field is the *stricter* (higher) of:

- the entity type's sensitivity, and
- the field's own sensitivity.

### Example

Suppose you have a "Company" entity type with sensitivity **Internal**, and it has three fields:

| Field | Field sensitivity | Effective sensitivity |
| --- | --- | --- |
| Company Name | Public | **Internal** (the entity type is stricter) |
| Revenue | Sensitive | **Sensitive** |
| Bank Account | Secret | **Secret** |

A user with **Internal** access would see only "Company Name". A user with **Sensitive** access would see "Company Name" and "Revenue". Only a user with **Secret** access would see all three fields. A user with **Public** access would not see the entity type at all.

## What happens when a field is hidden

Hidden fields are completely invisible to the user — they do not appear in any part of the interface:

- The data grid does not show the column.
- The new record form does not show the field.
- The record history page does not show changes to the field.
- The schema page and the [Data Dictionary](data-dictionary.md) do not list the field.
- The import column mapping does not offer the field.
- The [data stewardship](stewardship.md) views do not show the field's values.
- CSV exports do not include the column.
- [Natural language query](ai-query.md) cannot use the field.
- API responses do not include the field, and the API ignores it in request bodies.
- The [OData feed](odata.md) does not include the column, and leaves out an entity type entirely if none of its fields are visible.

Users cannot write to fields they cannot see. The field simply does not exist from their perspective. When they edit a record, the values of fields hidden from them are left unchanged.

## What happens when an entire entity type is hidden

If a user's access level is lower than the entity type's sensitivity, the entity type is hidden entirely:

- It does not appear in the **Schemas** list or the [Data Dictionary](data-dictionary.md).
- It does not appear in the data stewardship overview.
- It does not appear in the entity type dropdown when importing data.
- [Natural language query](ai-query.md) cannot use it.
- Opening its pages directly (for example from a bookmarked link) sends the user back to the entity types list.

The same applies to API keys: the entity type is left out of the API's schema listing and the OData feed, and requests for it return `403 Forbidden`.

## The overridden indicator

On the schema page, if a field's own sensitivity is lower than the entity type's sensitivity, the field's sensitivity badge is shown with a strikethrough. This indicates that the field's label is *overridden* by the entity type's stricter setting — a visual reminder that the effective sensitivity is higher than what the field alone specifies. The [Data Dictionary](data-dictionary.md#sensitivity-in-the-dictionary) uses the same indicator.

## Best practices

- Set the entity type sensitivity to the level appropriate for the *least sensitive* fields. Then raise individual field sensitivities for fields that need more protection.
- Give users the lowest access level they need. You can always raise it later.
- Create separate [API keys](api-keys.md) with different sensitivity levels for different integrations, rather than sharing a single high-clearance key.
- Remember that the admin *role* and the *access level* are independent. An admin with Internal access can manage entity types and users, but cannot see Sensitive or Secret fields or entity types. Admins who need to manage sensitive field configurations should have a correspondingly high access level.

---

[← Back to overview](../README.md)
