# Analysing a Parquet Export

The REST API can export an entity type as an [Apache Parquet](https://parquet.apache.org) file: one compact file with column types, which analysis tools read directly. In this tutorial you download the golden company records and query them with DuckDB, pandas and Polars.

**You need:** an [API key](../guide/api-keys.md), the Company data from [Your First Golden Record](first-golden-record.md), and at least one of the tools below.

## 1. Download the file

```bash
curl -s -G "$GREIN_URL/api/v1/company/export" \
  -H "Authorization: Bearer $GREIN_API_KEY" \
  --data-urlencode "source=merged" \
  -o company.parquet
```

`source=merged` gives you the golden records. Use `source=crm` for one source's values only, `since=` for only the records changed since a point in time, and any field name as a filter, for example `country=SE`. See [Export records as Parquet](../guide/api.md#export-records-as-parquet).

The file has the columns `_record_id`, `_created_at` and `_updated_at`, then one column per field, named after the field's API name. Amounts and quantities are text: the currency comes first (`SEK 38500000`) and the unit last (`12.5 kg`). Reference fields hold the referenced record's ID.

## 2a. Query it with DuckDB

[DuckDB](https://duckdb.org) queries a Parquet file in place, with SQL:

```sql
SELECT country, count(*) AS companies, sum(employees) AS employees
FROM 'company.parquet'
GROUP BY country
ORDER BY employees DESC;
```

Split an amount into currency and number:

```sql
SELECT company_name,
       split_part(annual_revenue, ' ', 1)                  AS currency,
       CAST(split_part(annual_revenue, ' ', 2) AS DOUBLE) AS revenue
FROM 'company.parquet'
WHERE annual_revenue IS NOT NULL;
```

## 2b. Load it into pandas

```python
import pandas as pd  # pip install pandas pyarrow

df = pd.read_parquet("company.parquet")
df[["currency", "revenue"]] = df["annual_revenue"].str.split(" ", expand=True)
df["revenue"] = pd.to_numeric(df["revenue"])
print(df.groupby("currency")["revenue"].sum())
```

## 2c. Load it into Polars

```python
import polars as pl  # pip install polars

df = pl.read_parquet("company.parquet")
print(df.filter(pl.col("employees") > 50).select("company_name", "city", "employees"))
```

## 3. Send a Parquet file back

Parquet works in the other direction too. Change something in pandas, save it and upload it to a source:

```python
df.loc[df["company_name"] == "Suomi Pixel Oy", "employees"] = 11
df[["org_number", "employees"]].to_parquet("changes.parquet", index=False)
```

```bash
curl -s "$GREIN_URL/api/v1/company/upload" \
  -H "Authorization: Bearer $GREIN_API_KEY" \
  -F "source=crm" -F "file=@changes.parquet"
```

Rows are matched on the unique key, `org_number`, so only the employee counts are updated. Columns that aren't fields, such as `_record_id`, are ignored. You can also drag a Parquet file onto the [Import data](../guide/importing-data.md#parquet-files) page.

## Next steps

- [Loading Grein Data into a Fabric Lakehouse](loading-into-a-fabric-lakehouse.md) uses the same export, incrementally.

---

[← Back to tutorials](README.md)
