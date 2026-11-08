# Fixing Incomplete Records

Source systems don't always send the unique key. In this tutorial a CRM sends an updated address for a company but leaves out its org number. You let Grein accept the row anyway, then attach it to the right company in the data stewardship view.

**You need:** the Company entity type and the CRM source from [Your First Golden Record](first-golden-record.md), the **admin** role (step 1) and the sample file [companies-crm-update.csv](data/companies-crm-update.csv).

## 1. Let the CRM send rows without an org number

By default, a row without a value for a unique field is rejected. Allow it for the CRM, flagged for review:

1. Open the **Company** schema page and click **Edit** next to **Org number**.
2. Click **Configure** in the **Source Key Overrides** section.
3. Pick **CRM** in the **— add source override —** dropdown.
4. Leave **Required** and **Unique** as they are and check **Defer Errors**. Save the field.

> **Screenshot:** The Source Key Overrides dialog for Org number, with a CRM row where Defer Errors is checked.
> <!-- screenshot: images/tutorial-fixing-incomplete-records-defer.png -->

## 2. Import the update

Import `companies-crm-update.csv` into **Company** with **CRM** as the destination. It has one row: Fjällvind Energi, without an org number, with a new city and phone number.

The preview counts the row as valid, because its only error is on a field with deferred errors. Click **Confirm import**.

The row couldn't be matched to an existing company, so it became a new, **incomplete** record. Incomplete records aren't merged, so it doesn't show up in Merged results.

## 3. Find it in data stewardship

Click **Fix data errors** in the sidebar. After a few seconds, the Company row shows **Incomplete: Match (1)**. Grein has compared the record with the existing companies in the background and found candidates.

> **Screenshot:** The stewardship overview with Incomplete: Match (1) and Deferred errors (1) on the Company row.
> <!-- screenshot: images/tutorial-fixing-incomplete-records-overview.png -->

## 4. Consolidate it into the right company

1. Click **Incomplete: Match (1)**, then **Review** on the Fjällvind Energi row.
2. The tray shows the incoming values and, below them, candidate records ranked by match score. **Fjällvind Energi AB** (556100-0002) should be at the top.
3. Click **Select →** next to it. The conflicts list shows the fields where the CRM already has a different value: city (Umeå or Luleå), phone and company name.
4. Keep the incoming city and phone, which are the new ones, and choose the existing company name, which is the full legal name.
5. Click **Consolidate**.

The values are moved to company 556100-0002 under the CRM source, and the incomplete record is deleted. Open Company in **Merged results**: Fjällvind Energi AB is now in Luleå.

> [!TIP]
> If you already know the missing key, there's a quicker way: click **Incomplete: Edit** on the stewardship overview and type the org number into the highlighted cell. The record then becomes complete and is matched like any other.

## Next steps

- [Data Stewardship](../guide/stewardship.md) covers potential duplicates and reference candidates, which you review the same way.
- [Deferred validation errors](../guide/validation-and-lookups.md#deferred-errors) explains which errors can be deferred.

---

[← Back to tutorials](README.md)
