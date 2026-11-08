# Asking Questions in Plain Language

The **Query** page turns a question in plain English into a query over your data. This tutorial gives you a set of questions to try on the sample company data, and shows how to rephrase when an answer isn't what you meant.

**You need:** the Company data from [Your First Golden Record](first-golden-record.md), and an AI model connected under **Settings > External AI Model** (see [Setup](../guide/ai-query.md#setup)). Any role can use the Query page, including viewers.

Only your question and the names and types of your entity types and fields are sent to the AI model. The record values stay in Grein.

## 1. Ask about the data

Click **Query** in the sidebar and try these, one at a time:

| Question | What you should get |
|---|---|
| *Which companies have more than 50 employees?* | Fjällvind Energi, Havn Logistik and Ljusdal Trä. |
| *Show companies in Sweden, sorted by name* | Fjällvind Energi, Kustbageriet, Ljusdal Trä and Nordlys Analytics, plus any Swedish companies you added in other tutorials. |
| *What is the average number of employees per country?* | One row per country. |
| *Which companies have no annual revenue?* | Suomi Pixel, which is only in the CRM, and any companies you added yourself. |

Click **Show SQL** under a result to see the query that was run and the values used in it.

> **Screenshot:** The Query page with "Which companies have more than 50 employees?" and the three-row result.
> <!-- screenshot: images/tutorial-query-results.png -->

## 2. Ask about a specific source

Queries use the golden records unless you say otherwise. Try:

*Show the ERP values for Nordlys Analytics*

The name comes back in capitals, as the ERP has it, rather than the CRM spelling used in the golden record.

## 3. Ask about the data model

Questions about structure are answered in text, without running a query:

- *What fields does Company have?*
- *Which field identifies a company?*

## 4. When the answer is off

The model only knows your field names and types, so vague questions can be read in more than one way. *Show the big companies* could mean many employees or high revenue. Be specific about the field and the threshold: *Show companies with an annual revenue above 50 million*.

Amounts in different currencies are compared by their number alone, without exchange rates, so a question about revenue across countries mixes SEK, DKK and NOK.

## Next steps

- [Natural Language Query](../guide/ai-query.md) in the guide lists the supported filters, aggregates and joins, and the limits.
- [Restricting Sensitive Fields](restricting-sensitive-fields.md): questions can only use fields the user is cleared to see.

---

[← Back to tutorials](README.md)
