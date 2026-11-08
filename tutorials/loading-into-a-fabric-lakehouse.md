# Loading Grein Data into a Fabric Lakehouse

In this tutorial you build a Microsoft Fabric notebook that copies the golden company records into a Delta table in a lakehouse, and on later runs loads only the companies that changed. It uses the REST API's Parquet export.

**You need:**

- A Fabric workspace with a lakehouse, and permission to create notebooks.
- An [API key](../guide/api-keys.md), ideally stored as a secret in Azure Key Vault.
- The Company data from [Your First Golden Record](first-golden-record.md).

> [!TIP]
> If you only want the data in a table without writing code, use a **Dataflow Gen2** with the [OData feed](../guide/odata.md#connecting-from-microsoft-fabric) instead. A notebook gives you more control over incremental loads.

## 1. Create the notebook

Create a notebook, attach it to your lakehouse and paste this into the first cell. Fill in your Grein address and the Key Vault details.

```python
import os
from datetime import timedelta

import requests
from delta.tables import DeltaTable
from pyspark.sql import functions as F

GREIN_URL = "https://<your Grein address>"
API_KEY = notebookutils.credentials.getSecret("https://<your-vault>.vault.azure.net/", "grein-api-key")
ENTITY = "company"
TABLE = f"grein_{ENTITY}"

# Only fetch records changed since the last load (minus 5 minutes of overlap).
params = {"source": "merged"}
if spark.catalog.tableExists(TABLE):
    last = spark.table(TABLE).agg(F.max("_updated_at")).first()[0]
    if last is not None:
        params["since"] = (last - timedelta(minutes=5)).isoformat() + "Z"

r = requests.get(f"{GREIN_URL}/api/v1/{ENTITY}/export", params=params,
                 headers={"Authorization": f"Bearer {API_KEY}"}, timeout=300)
if r.status_code == 404:
    print("No changes")
else:
    r.raise_for_status()
    # Save the file in the lakehouse, then let Spark read it with its own column types.
    os.makedirs("/lakehouse/default/Files/grein", exist_ok=True)
    with open(f"/lakehouse/default/Files/grein/{ENTITY}.parquet", "wb") as f:
        f.write(r.content)
    df = spark.read.parquet(f"Files/grein/{ENTITY}.parquet")

    if not spark.catalog.tableExists(TABLE):
        df.write.format("delta").saveAsTable(TABLE)
    else:
        (DeltaTable.forName(spark, TABLE).alias("t")
            .merge(df.alias("s"), "t._record_id = s._record_id")
            .whenMatchedUpdateAll()
            .whenNotMatchedInsertAll()
            .execute())
    print(f"Loaded {df.count()} records")
```

## 2. Run it

The first run creates the table `grein_company` with every company. Run it again: it prints *No changes*, because the export returns `404` when nothing matches. Change a company in Grein, wait a few seconds for the merge, and run it once more: only that company is loaded and updated in the table.

> **Screenshot:** The Fabric notebook output "Loaded 7 records" next to the grein_company table in the lakehouse explorer.
> <!-- screenshot: images/tutorial-fabric-notebook.png -->

## 3. Schedule it

Add the notebook to a data pipeline, or use the notebook's **Schedule** setting, to run it as often as you need.

## Things to know

- **The watermark is the table itself.** The script reads the highest `_updated_at` already loaded, so there's no state to keep elsewhere. Spark returns it without a time zone, in the session time zone (UTC unless you've changed it), which is why the script adds `Z`.
- **The last file is kept** under `Files/grein` in the lakehouse, which helps when you need to check what a run loaded.
- **Deletions aren't seen.** Deleted companies don't appear in an incremental export. Drop the table now and then (for example weekly) to force a full reload.
- **New fields need a full reload.** When a field is added to the entity type, the export gets a new column. Drop the table, or enable schema evolution for the merge, so the new column is added.
- **Large tables.** The export is a single file. For entity types with millions of records, the first full load can take a while; later incremental loads are small.

## Next steps

- [Analysing a Parquet Export](analysing-a-parquet-export.md) describes the columns in the file.
- [Keeping Another System in Sync](keeping-another-system-in-sync.md) explains the watermark pattern.

---

[← Back to tutorials](README.md)
