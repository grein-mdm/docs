# Tutorials

Short, step-by-step walkthroughs of common tasks in Grein. Each one gets you to a working result and then points to the [guide](../README.md#guide) for the details.

## Sample data

Most tutorials use the same small dataset: six companies from a CRM and six from an ERP, which overlap and disagree with each other. Start with [Your First Golden Record](first-golden-record.md), which sets it up; the other tutorials can then be followed in any order.

| File | Used in |
|---|---|
| [companies-crm.csv](data/companies-crm.csv) | Your First Golden Record |
| [companies-erp.csv](data/companies-erp.csv) | Your First Golden Record, Scheduling a Nightly File Upload |
| [companies-crm-update.csv](data/companies-crm-update.csv) | Fixing Incomplete Records |
| [companies-messy.csv](data/companies-messy.csv) | Cleaning Data on the Way In |

## Getting started

- [Your First Golden Record](first-golden-record.md): an entity type, two sources, two imports and the merged result.
- [Fixing Incomplete Records](fixing-incomplete-records.md): accept rows without a unique key and attach them to the right record.

## Data quality and enrichment

- [Cleaning Data on the Way In](cleaning-data-on-the-way-in.md): transformations and validation rules.
- [Enriching Records from an External API](enriching-records-from-an-api.md): fill in fields from a web service.
- [Extracting Attributes with AI](extracting-attributes-with-ai.md): turn free text into structured fields.
- [Asking Questions in Plain Language](asking-questions-in-plain-language.md): natural language query.

## REST API

- API quickstart in [Python](api-quickstart-python.md), [Node.js](api-quickstart-nodejs.md), [PowerShell](api-quickstart-powershell.md), [C#](api-quickstart-csharp.md) or [curl](api-quickstart-curl.md): list entity types, write records, read changes.
- [Keeping Another System in Sync](keeping-another-system-in-sync.md): incremental sync with a watermark.
- [Scheduling a Nightly File Upload](scheduling-a-nightly-upload.md): upload a CSV file every night with PowerShell and Task Scheduler.

## Parquet

- [Analysing a Parquet Export](analysing-a-parquet-export.md): download records and query them with DuckDB, pandas or Polars.
- [Loading Grein Data into a Fabric Lakehouse](loading-into-a-fabric-lakehouse.md): incremental loads into a Delta table.

## Reporting (OData)

- [Power BI with Incremental Refresh](power-bi-incremental-refresh.md)
- [Analysing Grein Data in Excel](analysing-in-excel.md)

## Governance

- [Restricting Sensitive Fields](restricting-sensitive-fields.md): sensitivity labels in the UI, the API and reports.
- [Answering an Auditor](answering-an-auditor.md): record history, change exports and the privileged action log.

---

[← Back to overview](../README.md)
