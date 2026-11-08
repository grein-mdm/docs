# Auditing

The platform keeps two audit trails: a **data change history** that tracks every field-level change to records, and a **privileged action log** that tracks administrative and security-relevant operations. Both can be exported as CSV files from **Settings > Audit**.

> [!NOTE]
> Audit exports require a plan that includes auditing. On other plans the **Export** button is disabled. The change history on each record's history page is available on every plan.

## Data change history

Every time a source's value for a field is set, changed or cleared, whether in the web interface, through the [REST API](api.md) or by an [import](importing-data.md), an entry is added to the change history. Each entry records:

- **Timestamp**: when the change was made (UTC).
- **User**: who made the change. Changes made through the API are attributed to the API key's own user (see [Configuring API Keys](api-keys.md#audit-attribution)).
- **Entity type** and **record ID**: which record was changed.
- **Field**: which field was changed.
- **Source**: which data source the value belongs to.
- **Value**: the new value. An entry without a value means the value was cleared.

Merged values are not recorded separately: they are recalculated from the source values, which are in the history.

The history is also visible per record in the web interface: click a record's ID in the data grid to open its history page. See [Viewing and Editing Data](viewing-and-editing.md#record-history).

## Privileged action log

Security-relevant operations made in the web interface are logged separately. Entries are only ever added; they can't be edited or removed. The following events are tracked:

| Category | Events |
|---|---|
| **authorization** | User invited, invitation revoked, user removed, role changed, sensitivity level changed. |
| **api_key** | API key created, rotated, enabled, disabled, deleted. |
| **schema** | Entity type created, updated, deactivated, activated, deleted. Field created, updated, removed. Source created, deleted. |
| **config** | AI settings updated or removed, subscription cancelled, workspace deactivated. |
| **data** | Data purged. |

Each log entry records:

- **Timestamp**: when the action occurred (UTC).
- **User**: the email address of the user who performed the action.
- **Category** and **Action**: what type of operation was performed, for example `authorization` and `role_changed`.
- **Target**: the type and ID of the affected object (for example a user, entity type, field or API key), where there is one.
- **Detail**: a JSON object with additional context, such as the new role, the sensitivity level or the AI provider and model.
- **IP address**: the IP address the request came from.

## Exporting audit data

Go to **Settings > Audit** and click **Export**. The **Audit Exports** dialog has two exports, each with its own **Download CSV** button:

- **Data change history**: the field-level change log.
- **Privileged action log**: the administrative action log.

> **Screenshot:** The Audit Exports dialog with the From and To date fields and a Download CSV button for each export.
> <!-- screenshot: images/audit-exports.png -->

Both have optional **From** and **To** dates; both dates are included in the export. Leave them blank to export everything. Rows are sorted newest first, and timestamps are in ISO 8601 format in UTC (for example `2026-04-05T14:23:01Z`).

The change history CSV has these columns: `Timestamp (UTC)`, `User ID`, `Entity Type`, `Record ID`, `Field`, `Field Name` (the machine name), `Source`, and one value column per kind of value: `Text Value`, `Number Value`, `Date Value` and `Bool Value`. Only the columns that fit the field type are filled in; amount and unit fields use both the text column (currency or unit) and the number column. `User ID` is the user's numeric ID.

The privileged action log CSV has these columns: `Timestamp (UTC)`, `User`, `Category`, `Action`, `Target Type`, `Target ID`, `Detail` and `IP Address`.

## Permissions

Audit exports are restricted to **admin** users, like the rest of the Settings page. Viewers and editors can't access them.

## Retention

The data change history is affected by two retention mechanisms:

- **Manual purge.** Under **Settings > Data storage**, click **Purge…**. Enter how many days of history to keep. Change history older than that is permanently deleted, together with deleted records, removed fields and other deleted data older than the same period. Enter 0 to delete all history and deleted data. A purge is recorded in the privileged action log.
- **Automatic retention policies.** When a [retention period](entities-and-fields.md#retention-policies) is set on an entity type or field, the platform applies it automatically once a day. Values that haven't been changed within the period are removed, together with their change history. If both the field and its entity type have a period, the shorter one applies. A record is removed only once it has no values left. Retention runs are not recorded in the privileged action log.

The privileged action log is **not affected by either mechanism**. It is kept indefinitely.

---

[← Back to overview](../README.md)
