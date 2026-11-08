# Power BI with Incremental Refresh

Connecting Power BI to the [OData feed](../guide/odata.md) takes a minute. For a large table, though, you don't want every scheduled refresh to download every record. In this tutorial you connect to the feed and set up incremental refresh, so that Power BI only reloads what has changed.

**You need:**

- Power BI Desktop, and a workspace in the Power BI service to publish to.
- An [API key](../guide/api-keys.md). Its sensitivity level decides which tables and columns Power BI can see.
- The Company data from [Your First Golden Record](first-golden-record.md), or any entity type of your own.

## 1. Connect to the feed

1. In Power BI Desktop, choose **Get data → OData feed**.
2. Enter `https://<your Grein address>/odata/v1` and click **OK**.
3. Choose **Basic**, enter `api` as the user name and paste the API key as the password.
4. Tick **company** and click **Transform Data**.

Every table has the columns `_record_id`, `_created_at`, `_updated_at` and `_version`, then one column per field.

## 2. Add the RangeStart and RangeEnd parameters

Incremental refresh needs two parameters with exactly these names.

1. In Power Query, choose **Manage Parameters → New Parameter**.
2. Create **RangeStart**, type **Date/Time**, current value `1/1/2020 00:00:00`.
3. Create **RangeEnd**, type **Date/Time**, current value `1/1/2030 00:00:00`.

## 3. Filter on the creation date

Select the **company** query. `_created_at` is a date/time *with time zone*, so instead of the column filter menu, add a step in the formula bar (**fx**):

```powerquery
= Table.SelectRows(company, each
    [_created_at] >= DateTime.AddZone(RangeStart, 0) and
    [_created_at] <  DateTime.AddZone(RangeEnd, 0))
```

Replace `company` with the name of the previous step if it's different. Use `>=` on one side and `<` on the other, so that no record falls into two partitions.

> [!WARNING]
> Filter on `_created_at`, not `_updated_at`. A record's creation date never changes, so it stays in the same partition. Partitioning on `_updated_at` would put a changed record in a second partition and count it twice.

Click **Close & Apply**.

## 4. Set the refresh policy

1. In the **Data** pane, right-click **company** and choose **Incremental refresh**.
2. Turn on **Incrementally refresh this table**.
3. **Archive data starting** 10 years before the refresh date, or however far back your records go.
4. **Incrementally refresh data starting** a period before the refresh date. See below for how to choose it.
5. Tick **Detect data changes** and choose **_updated_at**.
6. Click **Apply**.

> **Screenshot:** The Power BI incremental refresh dialog with archive and refresh periods set and Detect data changes on _updated_at.
> <!-- screenshot: images/tutorial-power-bi-incremental-refresh.png -->

Power BI often warns that it can't confirm whether an OData query can be folded. The filter above is a plain date comparison, which the feed supports, so the warning can be ignored.

### How far back to refresh

With **Detect data changes**, Power BI checks the highest `_updated_at` in each partition inside the refresh period, and reloads only the partitions where it has moved. Records *created* before the refresh period are never checked, so changes to them are missed until a full refresh.

So choose the refresh period by how far back records can still change. Master data like companies can change at any age, so a long period (several years) is usually right. Partitions without changes are skipped, so a long refresh period costs much less than a full reload.

## 5. Publish and schedule

1. Publish the report to your workspace.
2. In the Power BI service, open the semantic model's **Settings**. Under **Data source credentials**, edit the OData source, choose **Basic** and enter `api` and the API key.
3. Set up a **Scheduled refresh**.

The first refresh in the service loads everything and creates the partitions. Later refreshes load only the changes.

## When to do a full refresh

- **Deleted records** aren't removed by an incremental refresh. They disappear when their partition is reloaded.
- **New or removed fields** change the table's columns. Refresh the table in Power BI Desktop and publish again.

## Next steps

- [The OData Feed](../guide/odata.md) lists the supported query options and what to do when a step fails with "not supported".
- [Analysing Grein Data in Excel](analysing-in-excel.md) uses the same feed.

---

[← Back to tutorials](README.md)
