# Enriching Records from an External API

An API lookup calls a web service when a record is saved and copies values from the response into the record. In this tutorial you use the free [REST Countries](https://restcountries.com) service to fill in the region of each company's country.

**You need:** the Company entity type from [Your First Golden Record](first-golden-record.md), the **admin** role, and a plan that includes API lookups.

> [!NOTE]
> REST Countries is a free third-party service, used here because it needs no account. Grein has no control over it; if it's unavailable, try the same steps with any JSON API you have access to.

## 1. Add the target fields

On the **Company** schema page, add two **text** fields: **Region** (`region`) and **Subregion** (`subregion`).

## 2. Create the lookup

1. Click **API Lookups** in the sidebar, then **+ New lookup**.
2. **Display name:** *Region from country*.
3. **Entity type:** Company. **Trigger field:** Country.
4. **Method:** GET. **URL template:**

   ```text
   https://restcountries.com/v3.1/alpha/{value}?fields=region,subregion
   ```

   `{value}` is replaced by the record's country code, for example `SE`.

## 3. Test the call

Under **Test your API call**, enter `SE` and click **Send test**. The response looks something like this:

```json
{ "region": "Europe", "subregion": "Northern Europe" }
```

> **Screenshot:** The Test your API call panel with SE as the sample value and the JSON response.
> <!-- screenshot: images/tutorial-enriching-test.png -->

## 4. Map the response

Click **+ Add mapping** twice:

| JSON path | Target field |
|---|---|
| `region` | Region |
| `subregion` | Subregion |

If your test response was a list (it starts with `[`), use `0.region` and `0.subregion` instead.

Under **Result source**, choose **Create a new source** with the name *REST Countries* and slug `restcountries`. Save the lookup.

## 5. Trigger it

Lookups run when a record is saved in the web interface or through the [REST API](../guide/api.md). They don't run for the **Import data** page, so the companies you imported earlier aren't enriched yet.

Open **Company** under **Data** (the Manual entry source), and add a new company in the empty row at the bottom: org number `556100-0010`, name *Test Company AB*, country `FI`. When the row is saved, the lookup runs. Switch to the **REST Countries** source: the new company has region *Europe* and subregion *Northern Europe*. Shortly afterwards, the values are merged into the golden record too.

To enrich all existing records, send them through the API once, for example with an [upsert](../guide/api.md#upsert-create-or-update) of their org numbers and countries. See [API Quickstart: Python](api-quickstart-python.md).

## Next steps

- [API lookups](../guide/validation-and-lookups.md#api-lookups) in the guide covers POST requests, headers for authenticated APIs and timeouts.
- [Extracting Attributes with AI](extracting-attributes-with-ai.md)

---

[← Back to tutorials](README.md)
