# The OData Feed

The OData feed is a read-only connection to your master data, for Power BI, Microsoft Fabric, Excel and any other tool that can read an OData v4 feed. Every entity type becomes a table with one row per record and one column per field.

The feed address is `https://<your Grein address>/odata/v1`. It's available to every workspace with API access, and uses the same [API keys](api-keys.md) as the [REST API](api.md).

## What the feed contains

- **Merged records only.** Each table contains the consolidated ("merged") version of every record. Values held by individual sources are not included. See [Merging Data Sources](merging.md).
- **Only what your key may see.** Entity types and fields above the API key's [sensitivity level](sensitivity.md) are left out entirely, both from the data and from the table definitions.
- **Only active fields.** Fields that have been removed from an entity type are not included.

### Columns

Tables are named after the entity type's machine name.

| Column | Type | Description |
|---|---|---|
| `_record_id` | Whole number | The record's ID. This is the table's key. |
| `_created_at` | Date/time (UTC) | When the record was created. |
| `_updated_at` | Date/time (UTC) | When the record was last changed. See [Loading only changed records](#loading-only-changed-records). |
| `_version` | Whole number (64-bit) | A number that increases every time the record changes. Compare it with the value you stored last time to tell whether a record has changed. |
| *one per field* | see below | Named after the field's machine name. |

| Field type | Column type |
|---|---|
| Text, enum, lookup and custom types | Text |
| Number, amount, unit | Decimal number |
| Percent | Decimal number, as a fraction (25% is 0.25) |
| Date | Date/time with time zone (UTC). Date-only values appear as midnight. |
| Boolean | True/false |
| Reference | Whole number: the `_record_id` of the referenced record |

Amount and unit fields get an extra text column for the currency or unit. An amount field `price` becomes `price` and `price_currency`, and a unit field `weight` becomes `weight` and `weight_unit`. (If the entity type already has a field with that name, the extra column gets a double underscore instead, for example `price__currency`.)

Reference fields are plain ID columns. To link two tables in Power BI, create a relationship from the reference column to the other table's `_record_id`.

## Connecting from Power BI

1. In Power BI Desktop, choose **Get data → OData feed**.
2. Enter `https://<your Grein address>/odata/v1` and click **OK**.
3. When asked for credentials, choose **Basic**. Enter anything as the user name (for example `api`) and paste your API key as the password.
4. Select the tables you want and click **Load** or **Transform Data**.

> **Screenshot:** The Power BI Desktop OData feed credentials dialog with Basic selected, "api" as the user name and the API key in the password box.
> <!-- screenshot: images/odata-powerbi-credentials.png -->

Scheduled refresh in the Power BI service works the same way: set the data source's credentials to **Basic** with the API key as the password.

## Connecting from Microsoft Fabric

In a Dataflow Gen2, choose **Get data → OData** and connect the same way as in Power BI. In a data pipeline, create an **OData** connection with **Basic** authentication, using the API key as the password.

## Other tools and scripts

Any HTTP client can read the feed. Send the API key either as the Basic authentication password or as a Bearer token:

```http
Authorization: Bearer mdm_your_api_key_here
```

| Address | Returns |
|---|---|
| `/odata/v1` | The list of tables. |
| `/odata/v1/$metadata` | The table definitions (XML). |
| `/odata/v1/{entityType}` | The records of one entity type. |
| `/odata/v1/{entityType}({id})` | A single record. |
| `/odata/v1/{entityType}/$count` | The number of records, as plain text. Accepts `$filter`. |

Responses are JSON (`$format=json` is accepted; other formats are not). Only `GET` requests are allowed.

### Supported query options

| Option | Example |
|---|---|
| `$select` | `$select=sku,price,price_currency` |
| `$filter` | `$filter=country eq 'SE' and price gt 100` |
| `$orderby` | `$orderby=product_name desc` (several columns, comma-separated, are allowed) |
| `$top`, `$skip` | `$top=100&$skip=200` |
| `$count` | `$count=true` adds the total number of matching records (`@odata.count`). |

`$filter` supports `eq`, `ne`, `gt`, `ge`, `lt`, `le`, `and`, `or`, `not`, parentheses and `null`. Each comparison compares a column with a literal value. Text values go in single quotes (write `''` for a quote inside a value), and dates are written as `2026-01-31T00:00:00Z` or `2026-01-31`. A true/false column can be used on its own as a condition, for example `$filter=is_active` or `$filter=not is_active`.

The following are not supported and return `501 Not Implemented` rather than unfiltered data:

- Functions such as `contains()` or `startswith()`.
- Comparing two columns with each other.
- The query options `$expand`, `$search`, `$apply` and `$compute`.

**Power BI tip:** Power BI sends some filters you apply in Power Query to the server. If a refresh fails with "not supported", the step uses something the feed can't filter on (for example "Text contains"). Apply that step after a step that Power BI doesn't send to the server, such as **Table.Buffer**, or filter in the report instead.

### Paging

Results are returned in pages of up to 5,000 records, ordered by `_record_id` unless you use `$orderby`. When more records are available, the response contains an `@odata.nextLink` address for the next page. Power BI and Fabric follow it automatically. A client can ask for smaller pages with the `Prefer: odata.maxpagesize=1000` header.

### Concurrency

The feed is not subject to the REST API's [rate limit](api.md#rate-limiting), so a refresh that fetches many pages can't fail halfway through. Instead, each workspace can run at most two OData queries at a time. Further requests wait their turn rather than fail.

### Errors

Errors are returned as JSON in the OData format:

```json
{ "error": { "code": "NotImplemented", "message": "The function \"contains\" is not supported in $filter." } }
```

| Status | Meaning |
|---|---|
| `400` | Invalid query: unknown column, malformed `$filter`, wrong literal type, or unknown query option. |
| `401` | Missing, invalid or disabled API key. |
| `403` | The workspace doesn't have API access. |
| `404` | The table or record doesn't exist (or isn't visible to the key). |
| `405` | The request wasn't a `GET`: the feed is read-only. |
| `406` | A response format other than JSON was requested. |
| `501` | The query uses something the feed doesn't support (see above). |

<a id="changes"></a>

## Loading only changed records

Both `_updated_at` and `_version` change whenever anything in a record's row changes. That includes edits, imports and API writes, and also changes the platform makes on its own:

- The merged values being recalculated after a source changes. See [Merging Data Sources](merging.md).
- A [retention policy](entities-and-fields.md#retention-policies) removing expired values.
- A reference being cleared because the record it pointed to was deleted.

Filters on `_created_at`, `_updated_at` and `_version` are indexed, so a filter that matches few records is fast even on large tables.

### Power BI incremental refresh

Partition the table on `_created_at` (set up the `RangeStart`/`RangeEnd` filter on that column) and turn on **Detect data changes** with `_updated_at`. Don't partition on `_updated_at`: a record that changes would then end up in two partitions.

### Fabric pipelines and scripts

Store the highest `_updated_at` you have loaded, and next time request only records changed since then, for example `$filter=_updated_at ge 2026-11-01T08:00:00Z`. Then insert or update the rows by `_record_id` in your destination.

Start each load a few minutes *before* the stored value, for example 5 minutes, and let the update by `_record_id` remove the overlap. A change that was still being saved during your previous load can carry a slightly earlier timestamp than records you already received; the overlap makes sure you don't miss it.

### When you need a full reload

- **Deleted records** are not part of the feed, so incremental loads don't see deletions. Reload the table in full periodically, for example nightly or weekly, to remove them.
- **Schema changes.** After a field is added or removed, or a sensitivity change hides or reveals one, the table's columns change. Reload the table in full.

The feed is read-only. To create or change records, use the [REST API](api.md) or the web interface.

---

[← Back to overview](../README.md)
