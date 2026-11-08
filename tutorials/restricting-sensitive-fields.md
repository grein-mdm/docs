# Restricting Sensitive Fields

Not everyone who works with company data should see revenue figures. In this tutorial you label the revenue field as Sensitive and check that it disappears for users, API keys and reports that aren't cleared for it.

**You need:**

- The Company entity type from [Your First Golden Record](first-golden-record.md).
- The **admin** role, with access level **Sensitive** or **Secret** yourself (otherwise the field disappears for you too).
- A plan that includes sensitivity labels.

## 1. Label the field

1. On the **Company** schema page, click **Edit** next to **Annual revenue**.
2. Click **Configure** in the **Data Governance** section and set **Sensitivity** to **Sensitive**. Save.

The schema page now shows a Sensitive badge on the field. The entity type itself stays Internal, so everything else about a company is still visible to Internal users.

## 2. Check it with an API key

1. Under **Settings > API Access**, create a key labelled *Reporting* with sensitivity **Internal**.
2. Ask for one company with it:

   ```bash
   curl -s "$GREIN_URL/api/v1/company?pageSize=1" -H "Authorization: Bearer <the Reporting key>"
   ```

   The record has no `annual_revenue`. `GET /api/v1/entities` doesn't list the field either.
3. Run the same request with a key that has **Sensitive** or **Secret** level: the revenue is there.

A key that writes data can't change the hidden field either: a value for `annual_revenue` in a request body from the Reporting key is ignored, and the existing value is kept.

## 3. Check it in a report

Connect Excel or Power BI to the [OData feed](../guide/odata.md) with the Reporting key, as in [Analysing Grein Data in Excel](analysing-in-excel.md). The **company** table has no `annual_revenue` and `annual_revenue_currency` columns. They aren't empty; they don't exist for this key.

## 4. Check it for a user

Under **Settings > Users**, find a colleague with access level **Internal**, or invite one. When they open Company, the Annual revenue column isn't there: not in the grid, the CSV export, the record history, the data dictionary or the Query page. An editor with Internal access can still edit the other fields; the revenue values stay as they are.

## A plan for real data

- Set each entity type to the level of its *least* sensitive fields, and raise individual fields above it.
- Give people and API keys the lowest level they need. Create one key per integration, so each gets only what it needs.
- An API key's level can't be changed after it's created. To change it, create a new key and delete the old one.

## Next steps

- [Working with Sensitivity Labels](../guide/sensitivity.md) has the complete rules.
- [Answering an Auditor](answering-an-auditor.md): who changed what, and who gave whom access.

---

[← Back to tutorials](README.md)
