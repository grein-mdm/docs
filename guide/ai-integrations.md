# Building Integrations with AI Tools

AI coding assistants such as Claude Code, GitHub Copilot, Cursor or ChatGPT can write a working Grein integration for you, provided they know how your workspace's API works. This page shows how to give them that information, and how to keep your data safe while they work.

The short version: give the assistant your workspace's [OpenAPI description](api.md#openapi-descriptions), the [API guide](api.md), and a clear description of the job.

## What the assistant needs

An assistant writes better code from exact information than from guesses. Grein provides three things:

- **Your workspace's OpenAPI description** (`/api/v1/openapi.json`): every entity type, field, data type, required and unique field, and source, in a format assistants and code generators understand. This is the most important one.
- **The user documentation**, as plain Markdown. [The REST API](api.md) explains behaviour that a schema can't express, such as how upserts match records, how incremental sync works, and how to handle rate limits.
- **An index of the documentation for AI tools**, [`llms.txt`](https://raw.githubusercontent.com/grein-mdm/docs/main/llms.txt), which lists every page with a one-line summary. Assistants that can browse the web can start there and fetch the pages they need.

## Step by step

### 1. Create an API key and a source for the integration

Create a [new API key](api-keys.md#creating-an-api-key) for this integration only, with the lowest [sensitivity level](sensitivity.md) that covers the fields it needs. If the integration writes data, also create a [source](sources.md) for it, for example `erp_sync`. Writing to its own source keeps the integration's values apart from everything else, so you can see exactly what it wrote.

### 2. Download your workspace's description

```bash
curl -s "https://<your Grein address>/api/v1/openapi.json" \
  -H "Authorization: Bearer $GREIN_API_KEY" -o grein-openapi.json
```

Put `grein-openapi.json` in your project folder, or attach it to the conversation. The file contains your data model, not your data, and no key.

### 3. Describe the job

Tell the assistant what the integration should do, and point it to the description and the documentation. For example:

```text
Write a Python script that syncs customers from our ERP's CSV export into Grein.

- grein-openapi.json describes our Grein API. Use only the endpoints, fields and
  sources it lists.
- Read https://raw.githubusercontent.com/grein-mdm/docs/main/guide/api.md for how
  the API behaves (upsert, paging, errors, rate limits).
- Write to the source "erp_sync" with a batch upsert (PUT /api/v1/company).
- Map the CSV columns to the fields of CompanyUpsert; ask me if a column has no
  obvious match.
- Read the address from GREIN_URL and the key from GREIN_API_KEY. Never put the key
  in the code.
- On 429 or 503, wait the number of seconds in Retry-After and try again.
- Log rows that fail with 422, with their field errors, and carry on.
```

The more specific you are about the data, the better: name the entity type, the source, which fields to match on, and what should happen to rows that fail.

### 4. Try it on a small sample

Run the integration on a handful of records first. Open the entity type in Grein and pick the integration's source in the [source dropdown](viewing-and-editing.md#switching-sources) to check what it wrote, then switch to **Merged results** to see the golden records. When the result is right, run it on everything.

## Keep your data safe

> [!WARNING]
> Never paste an API key into a chat with an AI assistant. Keep it in an environment variable or a secret store, and have the generated code read it from there. If a key has been exposed, [rotate it](api-keys.md#rotating-a-key).

- **Give the integration its own key.** You can then disable it without affecting other integrations, and its changes are [attributed to that key](api-keys.md#audit-attribution) in the record history and audit log.
- **Use the lowest sensitivity level that works.** A key can't read or write fields above its level, and the OpenAPI description leaves them out, so the assistant never sees them.
- **Review the code before you run it**, as you would any code you didn't write. Look in particular at what it deletes: deleting a record's values for a source can't be undone through the API.
- **Write to a dedicated source.** The integration then can't overwrite values from other systems or from manual edits, and [survivorship rules](merging.md#survivorship-rules) decide how its values combine with the rest.

## When your data model changes

The OpenAPI description reflects your entity types and fields at the moment you download it. After adding, renaming or removing fields, download it again and ask the assistant to update the integration. Generated code that refers to a field that no longer exists will get `400` (filters) or have the value ignored (writes).

## Tips

- **Upsert rather than create.** `PUT /api/v1/{entityType}` creates new records and updates existing ones in one call, matched on the unique fields, so a sync can be run again safely. Ask for it explicitly.
- **For large loads, upload a file.** The [upload endpoint](api.md#upload-a-file) imports a CSV or Parquet file in one request.
- **For reading changes, use `since`.** See [Incremental sync](api.md#incremental-sync).
- **For reports, use OData.** Power BI and Fabric connect to the [OData feed](odata.md) directly; no code is needed.
- **Generate a client instead.** If you'd rather not have an assistant write the API calls, a code generator such as OpenAPI Generator can turn `grein-openapi.json` into a typed client library for most languages.

---

[← Back to overview](../README.md)
