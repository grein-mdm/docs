# Your First Golden Record

In this tutorial you load company data from two systems, a CRM and an ERP, that disagree with each other, and let Grein combine them into one golden record per company. It takes about 15 minutes.

**You need:** the **admin** role in a workspace. The two sample files: [companies-crm.csv](data/companies-crm.csv) and [companies-erp.csv](data/companies-erp.csv).

The other tutorials build on the entity type and sources you create here.

## 1. Create the entity type

1. Go to **Schemas** under **Configure** in the sidebar and click **+ New Entity Type**.
2. Enter **Company** as the display name. The slug becomes `company`.
3. Leave **Auto-merge** checked and click save.

## 2. Add the fields

On the schema page, click **+ Add Field** for each of these. Set the API name exactly as shown, since the sample files and the other tutorials use it.

| Display name | API name | Field type | Settings |
|---|---|---|---|
| Org number | `org_number` | text | **Unique** |
| Company name | `company_name` | text | |
| Country | `country` | Country (lookup) | |
| City | `city` | text | |
| Phone | `phone` | Phone Number (custom) | |
| Employees | `employees` | number | Decimal places: 0 |
| Annual revenue | `annual_revenue` | amount | |
| Description | `description` | text | |

`org_number` is the unique key: it's how Grein knows that a row from the CRM and a row from the ERP describe the same company.

> **Screenshot:** The Company schema page with all eight fields, Org number marked as unique.
> <!-- screenshot: images/tutorial-first-golden-record-schema.png -->

## 3. Create two sources

1. Go to **Sources** under **Configure**.
2. Add a source with the name **CRM** and slug `crm`.
3. Add a source with the name **ERP** and slug `erp`.

New sources are added at the bottom, so CRM now ranks above ERP. When the two disagree, the CRM value wins.

## 4. Import the CRM data

1. Click **Import data** in the sidebar.
2. Choose **Company** as the entity type and **CRM** as the destination.
3. Drag `companies-crm.csv` onto the text area.
4. All columns are matched automatically. Click **Preview**: you should see 6 valid rows.
5. Click **Confirm import**.

Notice that the country column says `Sweden`, `Denmark` and so on. The Country field stores the ISO code (`SE`, `DK`), and the names are translated as they come in.

## 5. Import the ERP data

Repeat step 4 with `companies-erp.csv` and **ERP** as the destination. This file uses country codes, has the company names in a different style, and has revenue figures that the CRM doesn't have. It also contains one company, Ljusdal Trä AB, that isn't in the CRM.

## 6. Look at the golden records

Open **Company** under **Data** in the sidebar and switch the source dropdown to **Merged results**. If some rows still have an amber left border, the merge hasn't run yet: wait a few seconds and reload, or click **Merge**.

| Company | What happened |
|---|---|
| Nordlys Analytics AB | Name and employee count (42) come from the CRM, which has priority. The revenue comes from the ERP, which is the only source that has it. |
| Suomi Pixel Oy | Only in the CRM, so it has no revenue. |
| Ljusdal Trä AB | Only in the ERP, so it has no city or phone. |

Each company is one record, combined from whatever each source knows about it. Click a record's ID to open its history: you can see which values each source contributed.

> **Screenshot:** The Company data grid showing Merged results, with Nordlys Analytics AB having a CRM name and an ERP revenue.
> <!-- screenshot: images/tutorial-first-golden-record-merged.png -->

## 7. Choose how conflicts are resolved

Source priority is only one way to resolve conflicts. Each field has its own **Survivorship Rule**, under **Data Processing** on the field's edit page: for example **Most Recent** for a phone number that changes often, or **Average** for a number that both systems estimate. See [Survivorship rules](../guide/merging.md#survivorship-rules).

> [!NOTE]
> A new survivorship rule or source order is used for each record the next time one of its values changes. Golden records that are already merged aren't recalculated straight away, so decide on the rules before you load large amounts of data.

## Next steps

- [Fixing Incomplete Records](fixing-incomplete-records.md): what happens when a source sends a row without the unique key.
- [Cleaning Data on the Way In](cleaning-data-on-the-way-in.md): transformations and validation rules.
- [Merging Data Sources](../guide/merging.md) and [Working with Sources](../guide/sources.md) in the guide.

---

[← Back to tutorials](README.md)
