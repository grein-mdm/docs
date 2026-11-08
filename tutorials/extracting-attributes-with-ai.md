# Extracting Attributes with AI

The CRM sample data has a free-text description for each company. In this tutorial you use an AI model to read those descriptions and fill in two structured fields: the industry and the year the company was founded.

**You need:**

- The Company entity type and the CRM data from [Your First Golden Record](first-golden-record.md).
- A plan that includes the external AI features, and an AI model connected under **Settings > External AI Model** (see [Setup](../guide/ai-query.md#setup)).
- The **editor** or **admin** role. Creating the fields and the source in step 1 needs **admin**.

> [!WARNING]
> The descriptions are sent to the AI provider your workspace has connected. That's fine for this sample data. Check your data policies before you do the same with real data.

## 1. Prepare the fields and a source

1. On the **Company** schema page, add a **text** field **Industry** (`industry`) and a **number** field **Founded year** (`founded_year`) with 0 decimal places.
2. Under **Sources**, add a source **AI extraction** (`ai_extraction`) and leave it at the bottom of the list.

Keeping AI output in its own source means you can always tell it apart from imported data. Because the source is last in priority, its values only fill gaps and never override a value from the CRM or the ERP.

## 2. Describe what to extract

Click **Attribute extraction** under **Tools** in the sidebar and fill in:

| Setting | Value |
|---|---|
| Entity type | Company |
| Input source | CRM |
| Input field | Description |
| Output source | AI extraction |
| Output fields | Industry, Founded year |
| Extraction instructions | *Extract the company's industry as a short English term (for example "Software" or "Logistics") and the year the company was founded.* |

## 3. Preview

Click **Preview**. Up to five descriptions are sent to the model, and the table shows what it extracted, for example:

| Description | Industry | Founded year |
|---|---|---|
| Nordlys Analytics was founded in 2014 and builds reporting dashboards… | Software | 2014 |
| Family-run bakery started in 1987… | Bakery | 1987 |

If the results aren't what you want, for example industries that are too specific, adjust the instructions and preview again. Each preview uses tokens on your AI provider account.

> **Screenshot:** The extraction preview table with descriptions, industries and founding years.
> <!-- screenshot: images/tutorial-extracting-preview.png -->

## 4. Run it

Click **Run extraction**. With six descriptions it finishes in seconds. Switch the Company data grid to the **AI extraction** source to see the results, and to **Merged results** to see them in the golden records.

Running the same extraction again only processes descriptions that haven't been processed yet, so it's cheap to rerun after new companies arrive. To redo everything with better instructions, click **Reset processed rows** first.

## Next steps

- [AI Attribute Extraction](../guide/ai-extraction.md) in the guide covers saved configurations, failures and limitations.
- [Asking Questions in Plain Language](asking-questions-in-plain-language.md)

---

[← Back to tutorials](README.md)
