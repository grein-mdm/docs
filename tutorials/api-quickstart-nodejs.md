# API Quickstart: Node.js

A short Node.js script that uses the [REST API](../guide/api.md) to list your entity types, write two companies and read back what changed. The same script is available for [Python](api-quickstart-python.md), [PowerShell](api-quickstart-powershell.md), [C#](api-quickstart-csharp.md) and [curl](api-quickstart-curl.md).

**You need:**

- Node.js 18 or later (it has `fetch` built in, so no packages are needed).
- An [API key](../guide/api-keys.md). Creating one needs the **admin** role, and your plan must include API access.
- The Company entity type and the CRM source from [Your First Golden Record](first-golden-record.md), or your own; adjust the names in the script.

## 1. Set your address and key

```bash
export GREIN_URL="https://<your Grein address>"
export GREIN_API_KEY="mdm_..."
```

## 2. The script

Save this as `grein-quickstart.mjs`:

```js
const BASE = process.env.GREIN_URL.replace(/\/$/, '') + '/api/v1';
const headers = { Authorization: `Bearer ${process.env.GREIN_API_KEY}` };

// Send a request, waiting and retrying if the rate limit is hit.
async function call(method, path, { query, body } = {}) {
  const url = new URL(BASE + path);
  for (const [k, v] of Object.entries(query ?? {})) url.searchParams.set(k, v);
  for (let attempt = 0; attempt < 5; attempt++) {
    const res = await fetch(url, {
      method,
      headers: body ? { ...headers, 'Content-Type': 'application/json' } : headers,
      body: body ? JSON.stringify(body) : undefined,
    });
    if (res.status === 429) {
      const wait = Number(res.headers.get('Retry-After') ?? 10);
      await new Promise(r => setTimeout(r, wait * 1000));
      continue;
    }
    if (!res.ok) throw new Error(`${res.status}: ${await res.text()}`);
    return res.json();
  }
  throw new Error('Still rate-limited after 5 attempts');
}

// 1. Which entity types and fields can this key see?
const { data: entityTypes } = await call('GET', '/entities');
for (const et of entityTypes) console.log(et.slug, et.fields.map(f => f.slug));

// 2. Create or update two companies in the CRM source, matched on org_number.
const started = new Date();
const results = await call('PUT', '/company', {
  query: { source: 'crm' },
  body: [
    { org_number: '556100-0001', company_name: 'Nordlys Analytics AB', employees: 44 },
    { org_number: '556100-0011', company_name: 'Granskog Data AB', country: 'SE' },
  ],
});
for (const row of results) console.log(row.error || row.errors ? 'error' : 'ok', row);

// 3. Read the golden records that changed since just before the write.
const since = new Date(started.getTime() - 5 * 60_000).toISOString();
for (let page = 1; ; page++) {
  const res = await call('GET', '/company', {
    query: { source: 'merged', since, page, pageSize: 200 },
  });
  for (const rec of res.data) console.log(rec.id, rec.company_name, rec.employees);
  if (page >= res.totalPages) break;
}
```

Run it with `node grein-quickstart.mjs`.

## 3. What happens

1. **`GET /entities`** lists the entity types and field names the key is cleared to see.
2. **`PUT /company`** with an array is a batch upsert. Nordlys Analytics already exists, so its CRM employee count is updated; Granskog Data is new and is created. Each row gets its own result, so one bad row doesn't stop the others.
3. **`GET /company?source=merged&since=…`** returns golden records changed since the given time, page by page.

> [!NOTE]
> Merging runs in the background, a few seconds after a write. If step 3 doesn't show the new values yet, run it again a moment later, or call `POST /company/merge` first to merge straight away.

## Next steps

- [Keeping Another System in Sync](keeping-another-system-in-sync.md) turns step 3 into a reliable sync.
- [The REST API](../guide/api.md) documents every endpoint, filter and error code.

---

[← Back to tutorials](README.md)
