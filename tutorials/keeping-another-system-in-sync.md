# Keeping Another System in Sync

Most integrations with Grein have the same shape: copy the golden records into another system (a data warehouse, a web shop, a reporting database) and keep that copy up to date. This tutorial shows the pattern with a short Python script that syncs companies into a local SQLite database. The same steps work in any language and for any destination.

**You need:** the [Python quickstart](api-quickstart-python.md) working, and the Company entity type from [Your First Golden Record](first-golden-record.md).

## The pattern

1. **Remember where you got to.** After each run, store the highest `updatedAt` you received. This is your *watermark*.
2. **Ask only for what changed.** Next time, request `?source=merged&since=<watermark minus 5 minutes>`. The overlap catches changes that were still being saved during the previous run.
3. **Write by ID.** Insert or update each record in the destination by its `id`. That way the overlap does no harm: a record you get twice is simply written twice.
4. **Reload in full now and then.** Deleted records don't show up in an incremental sync. A periodic full load, for example weekly, removes them from the destination.

The first run has no watermark, so it fetches everything: that's the full load.

## The script

```python
import json
import os
import sqlite3
from datetime import datetime, timedelta

import requests

BASE = os.environ["GREIN_URL"].rstrip("/") + "/api/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['GREIN_API_KEY']}"}
FULL_RELOAD = os.environ.get("FULL_RELOAD") == "1"

db = sqlite3.connect("companies.db")
db.execute("CREATE TABLE IF NOT EXISTS company (id INTEGER PRIMARY KEY, updated_at TEXT, data TEXT)")
db.execute("CREATE TABLE IF NOT EXISTS sync_state (name TEXT PRIMARY KEY, watermark TEXT)")

row = db.execute("SELECT watermark FROM sync_state WHERE name = 'company'").fetchone()
params = {"source": "merged", "pageSize": 200}
if row and not FULL_RELOAD:
    watermark = datetime.fromisoformat(row[0].replace("Z", "+00:00"))
    params["since"] = (watermark - timedelta(minutes=5)).isoformat()

seen, highest, page = set(), row[0] if row else None, 1
while True:
    r = requests.get(f"{BASE}/company", headers=HEADERS, params={**params, "page": page})
    r.raise_for_status()
    body = r.json()
    for rec in body["data"]:
        db.execute("INSERT OR REPLACE INTO company (id, updated_at, data) VALUES (?, ?, ?)",
                   (rec["id"], rec["updatedAt"], json.dumps(rec)))
        seen.add(rec["id"])
        highest = max(highest or rec["updatedAt"], rec["updatedAt"])
    if page >= body["totalPages"]:
        break
    page += 1

if "since" not in params:  # a full load: anything we didn't see has been deleted
    db.execute("DELETE FROM company WHERE id NOT IN (SELECT value FROM json_each(?))",
               (json.dumps(list(seen)),))

if highest:
    db.execute("INSERT OR REPLACE INTO sync_state (name, watermark) VALUES ('company', ?)", (highest,))
db.commit()
print(f"Synced {len(seen)} companies, watermark {highest}")
```

Run it a few times. The first run copies every company; later runs copy only what changed, often nothing. Run it with `FULL_RELOAD=1` to do a full load that also removes deleted companies.

Add the 429 handling from the [quickstart](api-quickstart-python.md) before you schedule it, so a busy period doesn't fail the run.

## Choosing between the API, Parquet and OData

| You want to… | Use |
|---|---|
| Sync into your own code or database | The REST API, as above. |
| Load large tables in one request, for example into a lakehouse | The [Parquet export](../guide/api.md#export-records-as-parquet), which takes the same `since` parameter. See [Loading Grein Data into a Fabric Lakehouse](loading-into-a-fabric-lakehouse.md). |
| Report on the data in Power BI, Excel or Fabric | The [OData feed](../guide/odata.md). See [Power BI with Incremental Refresh](power-bi-incremental-refresh.md). |

## Next steps

- [Incremental sync](../guide/api.md#incremental-sync) in the guide lists the changes that count as an update, including ones Grein makes by itself.

---

[← Back to tutorials](README.md)
