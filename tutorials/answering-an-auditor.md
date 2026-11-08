# Answering an Auditor

Two questions come up in every audit: *who changed this value, and when?* and *who has access, and who granted it?* This tutorial shows where Grein answers each one.

**You need:** the **admin** role for the exports, and a plan that includes auditing. The record history in step 1 is available to everyone, on every plan.

## 1. Who changed this value?

Say the auditor asks why Nordlys Analytics' employee count changed.

1. Open **Company** under **Data** and click the record's ID in the `#` column.
2. The history page shows the current golden values at the top, and below them every change, newest first, grouped by source: which fields changed, to what, when and by whom.

Changes made through the API are shown as "API — *key label*", which is why it pays to give each integration its own [API key](../guide/api-keys.md) with a clear label.

> **Screenshot:** The history page for Nordlys Analytics, with a change to employees from the CRM source attributed to "API — CRM sync".
> <!-- screenshot: images/tutorial-auditor-record-history.png -->

## 2. Export the change history for a period

For an auditor who wants the raw data:

1. Go to **Settings > Audit** and click **Export**.
2. Set **From** and **To** to the audit period. Both dates are included.
3. Click **Download CSV** under **Data change history**.

Each row is one change to one value: timestamp (UTC), user ID, entity type, record ID, field, source and the new value. An empty value means the value was cleared. Filter on `Record ID` and `Field Name` in Excel to follow one value through the period.

## 3. Who has access, and who granted it?

1. For the current state, open **Settings > Users > Configure**: every user with their role and access level. API keys and their sensitivity levels are under **Settings > API Access**.
2. For how it got that way, download the **Privileged action log** from the same Audit Exports dialog. It lists every invitation, role change, access-level change, user removal, API key change, schema change and data purge, with who did it, when, and from which IP address.

The privileged action log can't be edited or deleted, by anyone, and it's kept indefinitely.

## 4. Before you purge or set retention

Purging history (**Settings > Data storage**) and [retention policies](../guide/entities-and-fields.md#retention-policies) permanently delete change history along with the data. If you have to keep audit evidence for a number of years, export the change history before it's purged, and set retention periods no shorter than your obligation.

## Next steps

- [Auditing](../guide/audit.md) in the guide lists every event and CSV column.
- [Restricting Sensitive Fields](restricting-sensitive-fields.md)

---

[← Back to tutorials](README.md)
