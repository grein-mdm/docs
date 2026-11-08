# Analysing Grein Data in Excel

Excel can read the [OData feed](../guide/odata.md) directly, so you get a table of golden records that updates when you click **Refresh**, with no exporting and no copying. This tutorial takes about five minutes.

**You need:**

- Excel for Microsoft 365 on Windows. Excel for Mac has the same feature, but the menus are named differently.
- An [API key](../guide/api-keys.md). Ask an admin for one with the lowest sensitivity level you need.
- The Company data from [Your First Golden Record](first-golden-record.md), or any entity type of your own.

## 1. Connect

1. On the **Data** tab, choose **Get Data → From Other Sources → From OData Feed**.
2. Enter `https://<your Grein address>/odata/v1` and click **OK**.
3. In the credentials dialog, choose **Basic**, enter `api` as the user name and paste the API key as the password. Click **Connect**.

> **Screenshot:** Excel's OData feed credentials dialog with Basic selected.
> <!-- screenshot: images/tutorial-excel-credentials.png -->

## 2. Load a table

The Navigator lists one table per entity type the key can see. Select **company** and click **Load**. The companies appear as an Excel table, one row per company and one column per field.

To leave out columns or rows before loading, click **Transform Data** instead of **Load**, remove what you don't need, and click **Close & Load**.

## 3. Use it

The data is an ordinary Excel table: add a PivotTable (**Insert → PivotTable**), a chart, or `XLOOKUP` formulas against it. For example, a PivotTable with **country** in Rows and **employees** in Values shows the headcount per country.

## 4. Refresh

Click **Data → Refresh All** whenever you want the latest golden records. To refresh automatically, open **Data → Queries & Connections**, right-click the query, choose **Properties** and tick **Refresh data when opening the file**.

## Things to know

- **The key stays on your computer.** Excel saves the API key in your own data source settings, not in the workbook. A colleague who opens the workbook is asked for their own key.
- **What you see depends on the key.** Fields above the key's [sensitivity level](../guide/sensitivity.md) are left out.
- **It's read-only.** Changes you make in the Excel table don't go back to Grein, and are overwritten by the next refresh. To load changes, use [Import data](../guide/importing-data.md): you can copy cells from Excel and paste them on the import page.
- **Reference fields** are record IDs. Load both tables and use `XLOOKUP` on `_record_id`, or merge the queries in Power Query, to show the referenced record's name.

## Next steps

- [Power BI with Incremental Refresh](power-bi-incremental-refresh.md)

---

[← Back to tutorials](README.md)
