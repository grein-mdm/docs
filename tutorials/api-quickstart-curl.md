# API Quickstart: curl

The quickest way to try the [REST API](../guide/api.md): list your entity types, write two companies and read back what changed, straight from a terminal. The same steps are available as a script in [Python](api-quickstart-python.md), [Node.js](api-quickstart-nodejs.md), [PowerShell](api-quickstart-powershell.md) and [C#](api-quickstart-csharp.md).

**You need:**

- `curl`, which comes with macOS, Linux and Windows 10 or later. The examples use bash syntax; in Windows, run them in Git Bash or WSL.
- An [API key](../guide/api-keys.md). Creating one needs the **admin** role, and your plan must include API access.
- The Company entity type and the CRM source from [Your First Golden Record](first-golden-record.md), or your own; adjust the names in the commands.

## 1. Set your address and key

```bash
export GREIN_URL="https://<your Grein address>"
export GREIN_API_KEY="mdm_..."
```

## 2. List entity types and fields

```bash
curl -s "$GREIN_URL/api/v1/entities" \
  -H "Authorization: Bearer $GREIN_API_KEY"
```

The response lists every entity type the key can see, with its fields. Use the `slug` values in URLs and request bodies.

## 3. Create or update two companies

```bash
curl -s -X PUT "$GREIN_URL/api/v1/company?source=crm" \
  -H "Authorization: Bearer $GREIN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '[
        {"org_number": "556100-0001", "company_name": "Nordlys Analytics AB", "employees": 44},
        {"org_number": "556100-0011", "company_name": "Granskog Data AB", "country": "SE"}
      ]'
```

This is a batch upsert, matched on the unique field `org_number`. Nordlys Analytics already exists, so its CRM values are updated; Granskog Data is created. The response has one result per row.

## 4. Read what changed

```bash
curl -s -G "$GREIN_URL/api/v1/company" \
  -H "Authorization: Bearer $GREIN_API_KEY" \
  --data-urlencode "source=merged" \
  --data-urlencode "since=2026-11-08T09:00:00Z" \
  --data-urlencode "pageSize=200"
```

Set `since` to a time a few minutes before step 3. The response contains the golden records changed since then, with `page`, `total` and `totalPages`; add `page=2` and so on to get the rest. Pipe the output through [`jq`](https://jqlang.org) to make it readable.

> [!NOTE]
> Merging runs in the background, a few seconds after a write. If the new values aren't there yet, try again a moment later, or merge straight away with `curl -s -X POST "$GREIN_URL/api/v1/company/merge" -H "Authorization: Bearer $GREIN_API_KEY"`.

## 5. Download as Parquet

```bash
curl -s -G "$GREIN_URL/api/v1/company/export" \
  -H "Authorization: Bearer $GREIN_API_KEY" \
  --data-urlencode "source=merged" \
  -o company.parquet
```

See [Analysing a Parquet Export](analysing-a-parquet-export.md) for what to do with the file.

## Next steps

- [The REST API](../guide/api.md) documents every endpoint, filter and error code.
- Interactive API documentation is at `/api/docs` on your Grein address.

---

[← Back to tutorials](README.md)
