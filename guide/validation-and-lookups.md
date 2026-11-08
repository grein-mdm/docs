# Validation, Transformations, and API Lookups

Grein provides three layers of data quality control: **transformations** normalise incoming values, **validation rules** reject bad data, and **API lookups** enrich records from external services.

## Transformations

Transformations are processing steps applied to incoming field values *before* validation. They are configured per field on the [field edit page](entities-and-fields.md): click **Configure** in the **Data Processing** section and build the chain under **Transformations**. Add a step with the **+ add…** dropdown, drag steps to reorder them, and remove a step with its remove button.

Transformations run in order as a chain — the output of one step becomes the input to the next. The following transformations are available:

| Transformation | Effect | Example |
| --- | --- | --- |
| Trim | Remove leading and trailing whitespace. | `"  hello  "` → `"hello"` |
| UPPERCASE | Convert to uppercase. | `"hello"` → `"HELLO"` |
| lowercase | Convert to lowercase. | `"HELLO"` → `"hello"` |
| Title Case | Capitalise the first letter of each word (works with accented letters too). | `"hello world"` → `"Hello World"` |
| Whitespace → space | Replace every whitespace character (tabs, line breaks) with a space. | `"hello\tworld"` → `"hello world"` |
| Collapse spaces | Replace runs of spaces with a single space. | `"hello   world"` → `"hello world"` |
| Remove whitespace | Remove all whitespace characters. | `"hello world"` → `"helloworld"` |
| Keep numeric | Remove everything except digits and decimal points. A minus sign directly in front of the number is kept. | `"$1,234.56"` → `"1234.56"`, `"SEK -12"` → `"-12"` |
| Remove numeric | Remove all digits. | `"abc123"` → `"abc"` |
| First word | Keep only the first word. | `"John Smith"` → `"John"` |
| Last word | Keep only the last word. | `"John Smith"` → `"Smith"` |
| Unquote | Remove one pair of enclosing quotes or brackets: `"…"`, `'…'`, `[…]`, `(…)` or `<…>`. | `"\"hello\""` → `"hello"` |

Transformations are applied whenever values come in: when records are created or edited in the web UI, when data is imported, and when data is written through the [REST API](api.md). Empty values are skipped, and so are amount and quantity values (which consist of two parts).

> [!TIP]
> A hyphen that is part of a code is not treated as a minus sign by **Keep numeric**: `"INV-2024"` becomes `"2024"`, not `"-2024"`.

> **Screenshot:** The Data Processing dialog of a field, showing a transformation chain such as Trim → Collapse spaces → Title Case.
> <!-- screenshot: images/validation-and-lookups-transformation-chain.png -->

### Common transformation chains

- Clean and normalise a name: Trim → Collapse spaces → Title Case
- Normalise an identifier: Trim → UPPERCASE → Remove whitespace
- Extract a number from messy input: Trim → Keep numeric

## Validation rules

Validation rules are constraints that incoming values must satisfy. If a value fails validation, the record is rejected with a descriptive error message (unless the error is [deferred](#deferred-validation-errors)). Rules are configured per field on the [field edit page](entities-and-fields.md): click **Configure** in the **Format** section and fill in the inputs under **Validation Rules**. Leave an input empty to not apply that rule.

The available rules depend on the [field type](entities-and-fields.md#field-types):

### Text fields

| Rule | Description |
| --- | --- |
| Min Length | Minimum number of characters. |
| Max Length | Maximum number of characters. |
| Regex Pattern | A regular expression that the value must match, e.g. `^[A-Z]{2}$` for a two-letter country code. The match is case-sensitive. |
| Custom Error Message | The error message shown instead of the default one when the value doesn't match the pattern (or when a required value is missing). |

### Number and percent fields

| Rule | Description |
| --- | --- |
| Min Value / Min % | Minimum allowed value. |
| Max Value / Max % | Maximum allowed value. |
| Decimal Places | Number of decimals. Incoming values are rounded to this many decimals, and values are displayed with this precision. |

Percent fields accept values such as `12.5`, `12,5` and `12.5%`; the value is the percentage itself, so `12.5` means 12.5 %.

### Date fields

| Rule | Description |
| --- | --- |
| Earliest Date | The earliest date allowed. |
| Latest Date | The latest date allowed. |

Dates must be valid calendar dates in `YYYY-MM-DD` format.

### Enum fields

| Rule | Description |
| --- | --- |
| Allowed Values | A comma-separated list of valid values, e.g. `active, inactive, pending`. Incoming values must match one of them exactly, including upper and lower case. |

### Amount and quantity fields

For amount and quantity fields you can set **Decimal Places**, which controls how many decimals are displayed. An amount needs a valid three-letter currency code (such as `USD`, `EUR` or `GBP`) and a numeric amount; a quantity needs a numeric value and a unit that belongs to the field's quantity type.

### Other field types

- **Boolean** fields accept true/false values (also `1`/`0`).
- **Reference** fields have their own settings for which entity type (and which unique field) they point to; see [Reference fields](entities-and-fields.md#reference-fields).
- **[Lookup](entities-and-fields.md#lookup-types)** fields (for example country or currency codes) accept only values from their predefined list. Known aliases are translated to the canonical value automatically, so "United States", "US" and "USA" can all resolve to the same code.
- **Custom** field types bring their own rule — a format (pattern) or a list of allowed values — so they have no **Format** section of their own.

### Required and unique values

Whether a field is **Required** or **Unique** is set in the field's **Identity** section, and can be overridden per source with [Source Key Overrides](entities-and-fields.md#source-key-overrides). See [Configuring Entity Types and Fields](entities-and-fields.md#identity) for how these constraints work.

<a id="deferred-errors"></a>

## Deferred validation errors

By default, a record that fails validation is rejected outright — the API returns a `422` error and nothing is saved, and an import skips the row. This protects data quality but can block the ingest pipeline when a source sends data that is incomplete or does not meet all validation rules.

**Deferred validation errors** let you accept invalid data without breaking the ingest process. When enabled for a field and source, errors on that field are deferred: the value is saved anyway (or the record is saved without it, if a required value is missing), and the record is flagged so a data steward can review and fix it later.

### How it works

1. Data arrives through the [REST API](api.md) (create, update, upsert or file upload) or through an [import](importing-data.md).
2. Validation runs as normal, including required and unique checks. If any fields fail, Grein checks whether **Defer Errors** is enabled for each failing field for the source the data is written to.
3. If all failing fields have deferral enabled, the record is saved. Each value that failed validation is flagged, and so is each required value that is missing.
4. If any failing field does *not* have deferral enabled, the record is rejected as usual (a `422` error from the API, or an invalid row in an import).

### Configuring deferred errors

Deferred errors are configured per field, per source. On the [field edit page](entities-and-fields.md), click **Configure** in the **Source Key Overrides** section, pick a source in the **— add source override —** dropdown, and check **Defer Errors** for that source.

Deferred errors can only be enabled for individual named sources — not as the field's default, and not for the built-in **Manual** and **Merged** [sources](sources.md). Source Key Overrides are not available for amount and quantity fields.

> [!NOTE]
> When you add a source override, its **Required** and **Unique** checkboxes are pre-filled with the field's default values. If you only want to enable deferred errors, leave them as they are — otherwise you may accidentally make the field not required for that source.

> **Screenshot:** The Source Key Overrides dialog, with one source row where Defer Errors is checked.
> <!-- screenshot: images/validation-and-lookups-defer-errors.png -->

### Reviewing and fixing deferred errors

Records with deferred validation errors appear on the [data stewardship](stewardship.md) page (**Fix data errors** in the sidebar). The stewardship overview shows the number of records with validation errors for each entity type. Click **Deferred errors** to open the data grid filtered to the affected records, with the cells that need correcting highlighted and editable. See [Fixing deferred validation errors](stewardship.md#fixing-deferred-validation-errors) for details.

## Unit conversion

Quantity fields (e.g. distance, weight) can be configured to automatically **convert incoming values to a target unit**. On the field edit page, click **Configure** in the **Format** section and choose a unit under **Convert incoming values to**. Choose **— no conversion (keep original unit) —** to store values in whatever unit they arrive in.

When set, all incoming values — regardless of their original unit — are converted to the target unit before they are stored. This ensures consistency across sources that use different units. For example, if a distance field is set to convert to `km`, a value of `100 mi` is stored as `160.9344 km`.

## API lookups

API lookups automatically enrich records by calling an external web service when a record is saved. They are not available on all plans; if your plan doesn't include them, **API Lookups** is shown with a lock icon in the sidebar.

### How lookups work

1. A record is created or updated — in the web UI, or through the [REST API](api.md) (including file uploads).
2. Grein checks whether the entity type has any active API lookups.
3. For each lookup, it takes the **trigger field** value from the submitted data. If the trigger field has no value, that lookup is skipped.
4. It calls the configured URL, with the trigger value inserted into the URL template.
5. It reads the JSON response and copies the mapped response values into entity fields.
6. The enriched values are stored under the lookup's [result source](sources.md), and feed into the [merge](merging.md) like any other source.

Lookups run right after the record has been saved, before the response is returned, so a slow external service can delay the save by up to the lookup's timeout. A failed lookup — a timeout, an error status, or a response that isn't JSON — is skipped and does not undo the save. Lookup results never trigger further lookups (no cascading).

> [!NOTE]
> Lookups are not triggered by imports in the web UI's **Import data** page.

### Creating an API lookup

Click **API Lookups** in the sidebar (admins only), then **+ New lookup**. The form has these sections:

#### Basic

- **Display name** — a descriptive label (e.g. "Timezone from Country Code").
- **Machine name** — a lowercase identifier (letters, digits and underscores), generated from the display name. It cannot be changed later.

#### Trigger

- **Entity type** — which entity type this lookup applies to.
- **Trigger field** — the field whose value is sent to the external service. The lookup fires whenever a record is saved with a value in this field.

#### Request

- **Method** — GET or POST.
- **URL template** — the URL to call, with `{value}` where the trigger field's value should go, for example `https://api.example.com/lookup?code={value}`. The value is URL-encoded.
- **Timeout (ms)** — how long to wait for a response: 100–30,000 ms, default 5,000.
- **Request headers** — optional HTTP headers, such as `Authorization: Bearer …`. Click **+ Add header** for each one.
- **Request body template (POST only)** — a JSON body with a `{value}` placeholder. The value is inserted as a quoted JSON string, so don't put quotes around the placeholder: write `{"phone": {value}}`, not `{"phone": "{value}"}`.

Only the first `{value}` in the URL template and in the body template is replaced.

#### Test your API call

Enter a sample trigger value and click **Send test** to see the raw JSON response from the external service, without saving any data. This is useful for checking the URL template and headers, and for seeing the structure of the response before you set up the mappings. Tests time out after at most 15 seconds.

> **Screenshot:** The "Test your API call" panel of an API lookup, showing a sample value and the JSON response.
> <!-- screenshot: images/validation-and-lookups-test-panel.png -->

#### Response mappings

Map values from the JSON response to entity fields. Click **+ Add mapping** for each one; at least one mapping is required. Each mapping has:

- **JSON path** — a dot-notation path into the response, e.g. `timezone.name` or `location.country`. Use numeric segments for array elements, e.g. `results.0.name` or `0.country`. If the path points to an array, its values are joined with commas (`, `).
- **Target field** — the entity field to write the value to.

#### Result source

Choose the [source](sources.md) that lookup results are stored under: **Use an existing source** (any source except **Manual** and **Merged**), or **Create a new source** by entering a source display name and slug.

### Managing lookups

The API Lookups page lists all lookups with their entity type, trigger field, target source, number of mappings and whether they are active. From there you can:

- **Edit** — change the configuration (except the machine name).
- **Activate or deactivate** — toggle the **Active** checkbox on the edit page. Inactive lookups do not fire.
- **Delete** — permanently remove a lookup.

While a lookup is active, its trigger field and target fields cannot be removed, and its result source cannot be deleted. Delete the lookup (or deactivate it) first.

> [!NOTE]
> Deleting a lookup does not delete its source or the data it produced. Previously enriched values remain on the records under the lookup's source.

---

[← Back to overview](../README.md)
