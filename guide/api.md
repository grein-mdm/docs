# The REST API

The REST API lets you read, write and manage records programmatically. All endpoints are under `/api/v1` and require an [API key](api-keys.md). The API is available to workspaces whose plan includes API access.

Interactive API documentation with request and response examples is available at `/api/docs` (Swagger UI).

To load data into Power BI, Microsoft Fabric or other reporting tools, use the read-only [OData feed](odata.md) instead. It uses the same API keys.

## Authentication

Include your API key as a Bearer token in the `Authorization` header:

```http
Authorization: Bearer mdm_your_api_key_here
```

All `POST` and `PUT` requests must send `Content-Type: application/json`. The only exception is [file upload](#upload-a-file), which uses `multipart/form-data`.

A missing, unknown or disabled key returns `401 Unauthorized`. A valid key for a workspace that doesn't have API access returns `403 Forbidden`.

## Schema endpoints

### List entity types and fields

```http
GET /api/v1/entities
```

Returns the active entity types with their fields. Only entity types and fields within the API key's [sensitivity clearance](sensitivity.md) are included; an entity type with no visible fields is left out.

```json
{
  "data": [
    {
      "slug": "company",
      "name": "Company",
      "description": null,
      "fields": [
        {
          "slug": "company_name",
          "name": "Company name",
          "description": null,
          "datatype": "text",
          "is_unique": true,
          "is_required": true
        }
      ]
    }
  ]
}
```

`slug` is the machine name you use in URLs and request bodies. The [data dictionary](data-dictionary.md) in the web interface shows the same names.

### List sources

```http
GET /api/v1/sources
```

Returns all data sources ordered by priority, each with `slug`, `name` and `priority`.

## Reading records

### List records

```http
GET /api/v1/{entityTypeName}
```

Returns a paginated list of records. Supports the following query parameters:

| Parameter | Default | Description |
|---|---|---|
| `page` | 1 | Page number (starts at 1). |
| `pageSize` | 50 | Records per page (1–200). |
| `source` | `merged` | Which source's values to return. With a source other than `merged`, only records that have values from that source are listed. An unknown source name returns `400`. |
| `since` | *none* | ISO 8601 timestamp. Only returns records created or changed after this time. See [Incremental sync](#incremental-sync). |

**Field filtering:** any other query parameter is treated as a filter on the field with that machine name. For example, `?country=US&status=active` returns only records where the `country` field equals "US" and the `status` field equals "active". Filters are exact matches. Dates are written as `YYYY-MM-DD` and booleans as `true`/`false` (or `1`/`0`). On an amount field the filter matches the currency code, and on a unit field it matches the unit. An unknown field name, a field outside the key's clearance, or a value that doesn't fit the field type returns `400 Bad Request`.

Response format:

```json
{
  "data": [
    {
      "id": 1,
      "createdAt": "2026-01-15T10:30:00.000Z",
      "updatedAt": "2026-03-19T14:22:00.000Z",
      "company_name": "Acme Corp",
      "country": "US"
    }
  ],
  "page": 1,
  "pageSize": 50,
  "total": 128,
  "totalPages": 3
}
```

Field values are returned under the field's machine name. Most types are plain JSON values; a few have a specific shape:

| Field type | Value |
|---|---|
| Amount | `{ "currency": "SEK", "amount": 1500 }` |
| Unit | `{ "unit": "kg", "value": 12.5 }` |
| Percent | The percentage as a number: 25% is `25`. |
| Reference | The ID of the referenced record. |

A field that has no value may be missing from the record object, so treat a missing field the same as `null`.

### Get a single record

```http
GET /api/v1/{entityTypeName}/{recordId}
```

Returns a single record by ID, in the same format as the items in the list. Supports the `source` query parameter.

### Export records as Parquet

```http
GET /api/v1/{entityTypeName}/export
```

Downloads all matching records as an Apache Parquet file. Supports the same `source`, `since` and field-filter parameters as the list endpoint. If no records match, the endpoint returns `404 Not Found`.

## Creating records

### Create a record

```http
POST /api/v1/{entityTypeName}
```

Creates a new record. The request body must include a `source` field naming the [source](sources.md) the data belongs to (it can't be `merged` or `manual`), plus the field values:

```json
{
  "source": "crm_salesforce",
  "company_name": "Acme Corp",
  "country": "US",
  "revenue": 1500000
}
```

Returns `201 Created` with the new record, including its `id`. The values in the response are the ones stored for that source.

Amount and unit fields take an object, for example `{ "currency": "EUR", "amount": 99.5 }` or `{ "unit": "kg", "value": 12.5 }`. A reference field accepts either the referenced record's ID or its unique key value, for example the referenced product's SKU.

[Transformations and validation rules](validation-and-lookups.md) are applied as in the web interface. If a value fails validation, the request returns `422` with a map of field errors, unless the field is set to [defer validation errors](validation-and-lookups.md#deferred-validation-errors).

### Upsert (create or update)

```http
PUT /api/v1/{entityTypeName}
```

Creates a new record or updates an existing one, matched by the entity type's [unique fields](entities-and-fields.md). Every unique field must be present in the body. If exactly one record matches, it is updated (fields left out of the body keep their current values). If no record matches, a new one is created. If several records match, the request fails with `409 Conflict`. An entity type without a unique field can't be upserted (`422`).

A single object returns `200 OK` (updated) or `201 Created` (created) with the record.

You can also send an **array** of objects to upsert many records in one call. Each row is processed on its own, and the response is an array with one result per row, in the same order: either the record, or an object with `error` or `errors` for a row that failed. The status is `200 OK` when every row succeeded and `207 Multi-Status` when at least one failed. All rows are written to the same source: give it as a `?source=` query parameter or in the first object.

This is the recommended endpoint for data integrations, as it handles both new and existing records in a single call.

### Upload a file

```http
POST /api/v1/{entityTypeName}/upload
```

Bulk-imports a CSV, TSV or other delimited text file, or an Apache Parquet file. Send it as `multipart/form-data` with the file in a `file` part and the source name in a `source` part (or as a `?source=` query parameter).

- For delimited files, the delimiter is detected automatically.
- Column headers are matched to field machine names, ignoring case. Columns that don't match a field, or that belong to a field outside the key's clearance, are ignored.
- Rows are matched to existing records by unique key, so existing records are updated and new ones created.
- Files can be up to 50 MB. Parquet files can have up to 100,000 rows; nested column types (arrays, maps, structs) are skipped. A Parquet file that is too large or can't be read returns `422` with the reason.

Valid rows are imported and invalid rows are skipped. The response reports what happened:

```json
{
  "created": 120,
  "updated": 8,
  "skipped": 1,
  "errors": [
    { "row": 42, "messages": ["Email must be a valid email address"] }
  ]
}
```

## Updating records

### Update by ID

```http
PUT /api/v1/{entityTypeName}/{recordId}
```

Updates an existing record's values for the source given in `source`. This is a **partial update**: fields not included in the request body keep their current values. Only the specified fields are overwritten.

## Deleting records

### Delete a record

```http
DELETE /api/v1/{entityTypeName}/{recordId}
```

Deletes the record's values for one source. The source is required, as a `source` query parameter or a `source` field in a JSON body. When no other source has values for the record, the record itself is deleted. Returns `{ "ok": true }`.

If other records reference this record, the request fails with `409 Conflict` and lists the referencing fields:

```json
{
  "error": "Record has references and cannot be deleted.",
  "dependents": [
    { "fieldId": 17, "fieldDisplayName": "Product", "entityTypeName": "Order", "count": 3 }
  ]
}
```

## Merging

### Trigger a merge

```http
POST /api/v1/{entityTypeName}/merge
```

Runs the merge for the specified entity type, consolidating values from all sources into the golden record, and returns `{ "ok": true }` when it has finished. See [Merging Data Sources](merging.md).

## Incremental sync

The `since` parameter on the list and export endpoints enables efficient incremental synchronisation. Pass the timestamp of your last sync, and only records that have been created or changed after it are returned. That includes changes the platform makes on its own: merged values being recalculated, [retention policies](entities-and-fields.md#retention-policies) removing values, and references being cleared because the referenced record was deleted.

A typical sync loop:

1. Fetch records with `?source=merged&since=2026-03-19T00:00:00Z`, following `page` until you reach `totalPages`.
2. Process the returned records, inserting or updating them by `id` in your destination.
3. Store the highest `updatedAt` you received, for the next sync.

Start each sync a few minutes *before* the stored timestamp, for example 5 minutes, and let the update by `id` absorb the overlap. A change that was still being saved during your previous sync can carry a slightly earlier timestamp than records you already received; the overlap makes sure you don't miss it.

> [!NOTE]
> Deleted records no longer appear in the API, so an incremental sync doesn't see deletions. Reload in full periodically to remove them from your destination.

## Error responses

The API uses standard HTTP status codes:

| Status | Meaning |
|---|---|
| `400` | Bad request: unknown filter field, malformed filter value, invalid `since` value, unknown `source` when reading, or no file in an upload. |
| `401` | Unauthorised: missing, invalid or disabled API key. |
| `403` | Forbidden: the workspace doesn't have API access, the key's sensitivity clearance doesn't cover any of the entity type's fields, the entity type is deactivated (read-only), or a plan limit on storage or records has been reached. |
| `404` | Not found: the entity type or record doesn't exist. |
| `409` | Conflict: ambiguous upsert match, or the record has dependents that prevent deletion. |
| `415` | Unsupported media type: write requests must use `Content-Type: application/json` (uploads: `multipart/form-data`). |
| `422` | Unprocessable: field validation errors, or a missing, unknown or reserved `source` when writing. |
| `429` | Too many requests: rate limit exceeded. See [Rate limiting](#rate-limiting). |

Error response format:

```json
{ "error": "Entity type \"foo\" not found." }
```

Validation error format, a map of field names to error messages:

```json
{
  "errors": {
    "email": "Email must be a valid email address",
    "name": "Name is required"
  }
}
```

## Rate limiting

API requests can be rate-limited per workspace. The limit is set by the platform administrator as a maximum number of requests per minute, shared by all API keys of the workspace. When a limit is set, every response carries the standard rate-limit headers ([IETF draft-7](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/07/)):

| Header | Example | Description |
|---|---|---|
| `RateLimit-Policy` | `100;w=60` | The limit and the length of the window in seconds. |
| `RateLimit` | `limit=100, remaining=42, reset=17` | The limit, the requests remaining in the current window, and the seconds until the window resets. |

When the limit is exceeded, the API returns `429 Too Many Requests` with a `Retry-After` header (in seconds) and a JSON error body:

```json
{ "error": "API rate limit exceeded. Please retry after the window resets." }
```

Use `remaining` and `reset` to implement backoff in your integration. If no rate limit is set for the workspace, requests are not throttled and these headers are omitted.

The [OData feed](odata.md) is not subject to this limit, so a Power BI refresh can't fail halfway through because it fetches many pages. Instead, each workspace can run at most two OData queries at a time; further requests wait their turn.

## Sensitivity

The API key's [sensitivity level](sensitivity.md) controls which entity types and fields are visible. Fields outside the key's clearance are left out of responses and ignored in request bodies; when a record is updated, their existing values are kept unchanged. If no fields of an entity type are visible, the API returns `403 Forbidden` for it.

---

[← Back to overview](../README.md)
