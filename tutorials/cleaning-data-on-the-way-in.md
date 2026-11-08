# Cleaning Data on the Way In

Data from other systems is rarely tidy: stray spaces, codes without their hyphen, countries written in the local language. In this tutorial you add transformations that clean values automatically and a validation rule that rejects values that can't be fixed, and you see both at work in an import preview.

**You need:** the Company entity type and the CRM source from [Your First Golden Record](first-golden-record.md), the **admin** role, and the sample file [companies-messy.csv](data/companies-messy.csv).

## 1. See what's wrong

Import `companies-messy.csv` into **Company** with **CRM** as the destination and click **Preview**, but don't confirm yet. Only one of the three rows is valid:

| Row | Problem |
|---|---|
| Kustbageriet | Phone number `031-555 03 03` has no country code. The Phone Number type expects international format, such as `+46 31 555 03 03`. |
| Östra Verkstaden | Valid, but the org number `5561000008` has no hyphen, unlike all the others. |
| Lilla Kaffet | `Sverige` isn't a recognised country name. |

Kustbageriet's name, `  Kustbageriet   AB `, also has extra spaces. It passes validation, but it would end up in the CRM as typed.

Leave the import page open in a separate tab.

## 2. Add transformations to the company name

Transformations run before validation, on every value that comes in: imports, the API and the web UI.

1. On the **Company** schema page, click **Edit** next to **Company name**.
2. Click **Configure** in the **Data Processing** section.
3. Under **Transformations**, add **Trim**, then **Collapse spaces**. Save.

> **Screenshot:** The Data Processing dialog for Company name with the Trim → Collapse spaces chain.
> <!-- screenshot: images/tutorial-cleaning-transformations.png -->

## 3. Add a validation rule to the org number

1. Click **Edit** next to **Org number**, then **Configure** in the **Format** section.
2. Under **Validation Rules**, set **Regex Pattern** to `^\d{6}-\d{4}$`.
3. Set **Custom Error Message** to *Org number must be written as 123456-7890.* Save.

## 4. Preview again

Go back to the import tab and click **Preview** again. Now:

- Kustbageriet's name is cleaned up, but the row is still rejected because of the phone number.
- Östra Verkstaden is rejected with your custom error message, instead of slipping through.
- Lilla Kaffet is still rejected for its country.

Fix the file: give Kustbageriet's phone number its country code (`+46 31 555 03 03`), add the hyphen to `556100-0008` and change `Sverige` to `Sweden` or `SE`. Paste the corrected data, preview and import. Kustbageriet matches the existing company, so its CRM values are updated. The other two are created as new companies.

> [!TIP]
> When you can't fix the source data before loading it, you can let the errors through for one source and fix them afterwards in data stewardship. See [Fixing Incomplete Records](fixing-incomplete-records.md) for how to set that up.

## What else is there

- Lookup fields like Country accept the code, the English name and common aliases (`SE`, `Sweden`, `SWE`) in any letter case, and always store the code.
- Number fields can have a minimum, a maximum and a number of decimal places. Quantity fields can convert every incoming value to one unit, such as kilograms.
- Enum fields accept only a fixed list of values.

See [Validation, Transformations, and API Lookups](../guide/validation-and-lookups.md) for every rule and transformation.

## Next steps

- [Enriching Records from an External API](enriching-records-from-an-api.md)
- [Extracting Attributes with AI](extracting-attributes-with-ai.md)

---

[← Back to tutorials](README.md)
