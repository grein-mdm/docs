# API Quickstart: Python

A short Python script that uses the [REST API](../guide/api.md) to list your entity types, write two companies and read back what changed. The same script is available for [Node.js](api-quickstart-nodejs.md), [PowerShell](api-quickstart-powershell.md), [C#](api-quickstart-csharp.md) and [curl](api-quickstart-curl.md).

**You need:**

- Python 3.9 or later and the `requests` package (`pip install requests`).
- An [API key](../guide/api-keys.md). Creating one needs the **admin** role, and your plan must include API access.
- The Company entity type and the CRM source from [Your First Golden Record](first-golden-record.md), or your own; adjust the names in the script.

## 1. Set your address and key

Keep the key out of your code by putting it in an environment variable:

```bash
export GREIN_URL="https://<your Grein address>"
export GREIN_API_KEY="mdm_..."
```

On Windows, use `setx GREIN_API_KEY "mdm_..."` and open a new terminal.

## 2. The script

Save this as `grein_quickstart.py`:

```python
import os
import time
from datetime import datetime, timedelta, timezone

import requests

BASE = os.environ["GREIN_URL"].rstrip("/") + "/api/v1"
session = requests.Session()
session.headers["Authorization"] = f"Bearer {os.environ['GREIN_API_KEY']}"


def call(method, path, **kwargs):
    """Send a request, waiting and retrying if the rate limit is hit."""
    for _ in range(5):
        r = session.request(method, BASE + path, **kwargs)
        if r.status_code != 429:
            r.raise_for_status()
            return r.json()
        time.sleep(int(r.headers.get("Retry-After", "10")))
    raise RuntimeError("Still rate-limited after 5 attempts")


# 1. Which entity types and fields can this key see?
for et in call("GET", "/entities")["data"]:
    print(et["slug"], [f["slug"] for f in et["fields"]])

# 2. Create or update two companies in the CRM source, matched on org_number.
started = datetime.now(timezone.utc)
results = call("PUT", "/company", params={"source": "crm"}, json=[
    {"org_number": "556100-0001", "company_name": "Nordlys Analytics AB", "employees": 44},
    {"org_number": "556100-0011", "company_name": "Granskog Data AB", "country": "SE"},
])
for row in results:
    print("error" if "error" in row or "errors" in row else "ok", row)

# 3. Read the golden records that changed since just before the write.
since = (started - timedelta(minutes=5)).isoformat()
page = 1
while True:
    body = call("GET", "/company", params={"source": "merged", "since": since,
                                           "page": page, "pageSize": 200})
    for rec in body["data"]:
        print(rec["id"], rec.get("company_name"), rec.get("employees"))
    if page >= body["totalPages"]:
        break
    page += 1
```

Run it with `python grein_quickstart.py`.

## 3. What happens

1. **`GET /entities`** lists the entity types and field names the key is cleared to see.
2. **`PUT /company`** with a list is a batch upsert. Nordlys Analytics already exists, so its CRM employee count is updated; Granskog Data is new and is created. Each row gets its own result, so one bad row doesn't stop the others.
3. **`GET /company?source=merged&since=…`** returns golden records changed since the given time, page by page.

> [!NOTE]
> Merging runs in the background, a few seconds after a write. If step 3 doesn't show the new values yet, run it again a moment later, or call `POST /company/merge` first to merge straight away.

## Next steps

- [Keeping Another System in Sync](keeping-another-system-in-sync.md) turns step 3 into a reliable sync.
- [The REST API](../guide/api.md) documents every endpoint, filter and error code.
- Interactive API documentation is at `/api/docs` on your Grein address.

---

[← Back to tutorials](README.md)
