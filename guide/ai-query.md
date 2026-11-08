# Natural Language Query

The natural language query feature lets you ask questions about your data in plain English. The platform translates your question into a structured query using an external AI model that your workspace connects, checks it against your schema, and returns the results.

> [!NOTE]
> Natural language query requires a plan that includes external AI. On other plans, **Query** is shown with a lock in the sidebar and the settings to connect a model are disabled.

## Setup

Before anyone can use queries, a workspace admin must connect an external AI model under **Settings > External AI Model**, by clicking **Configure**. The following providers are supported:

- OpenAI
- Anthropic
- Azure OpenAI
- Mistral
- Groq
- Together
- Cohere
- Custom (any OpenAI-compatible endpoint)

Fill in:

| Setting | Description |
|---|---|
| **Provider** | One of the providers above. |
| **API Key** | Your API key with that provider. It is stored encrypted and never shown again after saving. To keep the current key when changing other settings, leave the box blank. |
| **Base URL** | Optional. Overrides the provider's default endpoint. Required for Azure OpenAI and custom providers. |
| **Model** | Required. Click **Load models** to list the models your API key can use, or type a model ID. For Azure OpenAI, enter your deployment name. |
| **Requests/min**, **Input tokens/min**, **Output tokens/min** | Optional rate limits for calls to the AI provider. When a limit is reached, further queries wait until there is room again. Leave blank for unlimited. |

Click **Save AI settings**. To disconnect the model and delete the stored API key, click **Remove**. Saving and removing AI settings are recorded in the [privileged action log](audit.md#privileged-action-log).

> **Screenshot:** The External AI Model dialog in Settings, with a provider selected, the model list loaded and the rate-limit fields below.
> <!-- screenshot: images/ai-query-settings.png -->

## Using the query page

Click **Query** in the sidebar. Every member of the workspace can use it, including viewers. Type a question in the search bar and click **Query** (or press Enter).

> **Screenshot:** The Query page with a question in the search bar and a results table below, with the generated SQL shown.
> <!-- screenshot: images/ai-query-results.png -->

### Data questions

Ask about the contents of your data. Results are displayed in a table. Click **Show SQL** to see the generated query and its parameter values. Examples:

- "Show me all products with a price above 100"
- "Which countries have an area larger than Sweden?"
- "List vendors with more than 5 products, sorted by name"
- "What is the average price per country?"

### Metadata and structural questions

Ask about how the data model is structured. The AI answers in plain text based on your schema, and no query is run. Examples:

- "How do countries and currencies fit together?"
- "What entity types are available?"
- "What fields does the Product entity have?"
- "Which entities reference Company?"

The AI chooses the appropriate response type automatically based on your question.

## How it works

When you submit a question, the platform:

1. **Builds a schema description**: the machine names and types of the entity types and fields you have access to (filtered by your [sensitivity level](sensitivity.md)), including the relationships between entity types. This description is sent to the AI model together with your question. No record data is sent.
2. **Calls the AI model.** For data questions, the model returns a structured query specification in JSON. For metadata questions, it returns a plain-text explanation derived from the schema.
3. **Checks the response.** Every entity name, field name, operator and join in the specification is checked against your actual schema. Unknown names, fields you can't access, and malformed specifications are rejected.
4. **Builds the query.** The checked specification is turned into a database query over your data, with one column per field. All values are passed as parameters, never inserted into the query text.
5. **Runs it and returns the results.** The query runs with a 30-second timeout and returns at most 1,000 rows (50 if the question doesn't ask for a particular number).

## Default behaviour

By default, queries return **merged (golden record) values only**. If you want to see values from a specific source, mention it in your question (for example "show me the CRM values for…").

## Joins and references

Queries can combine entity types when a **reference field** links them. A reference field on one entity type points to records in another entity type. The AI model is aware of these relationships and can use them to answer questions that span several entity types.

For example, if "Product" has a reference field "vendor" pointing to "Vendor", you can ask "list products with their vendor name" and the platform will join the two entity types.

A query can have at most 3 joins, so it can combine up to four entity types.

## Comparative queries

You can compare values against other records. For example, "show me countries with an area larger than Sweden" looks up Sweden's area and uses it as the filter value. The AI model expresses this as a structured reference to the other record, which the platform checks and runs like any other condition.

## Filters and aggregates

Conditions can use `=`, `!=`, `>`, `<`, `>=`, `<=`, `in`, `not in`, `like`, `is null` and `is not null`.

The following aggregate functions are supported: `count`, `sum`, `avg`, `min` and `max`. `sum` and `avg` can only be used on numeric fields.

## Security

The query feature treats the AI model's output as untrusted:

- **No SQL from the AI.** The model produces a JSON specification, not SQL. The platform builds the query itself.
- **Schema allow-list.** Every entity name, field name and operator in the specification is checked against your workspace's actual schema. Unknown names are rejected.
- **Parameterised values.** All filter values are passed as parameters, never inserted into the query text.
- **Sensitivity filtering.** The schema sent to the AI, and the fields available in queries, are limited to your sensitivity level. Fields above it are never exposed.
- **No record data sent to the AI.** Only the schema (entity type and field names, field types and relationships) and your question are sent to the AI model. Record values never leave the platform, so avoid putting confidential data in the question itself.
- **Encrypted credentials.** The AI provider API key is stored encrypted and can't be read back from the settings page.

## Limitations

- Results are capped at 1,000 rows.
- Queries time out after 30 seconds.
- A maximum of 3 joins per query.
- The AI model may occasionally misinterpret ambiguous questions. If the results don't look right, try rephrasing your question more specifically.
- The quality of results depends on the AI model your admin has connected.

---

[← Back to overview](../README.md)
