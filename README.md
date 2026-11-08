# Grein Documentation

Grein is a master data management platform that helps you collect, consolidate, and maintain a single source of truth across your organisation. It brings together data from multiple systems, lets you define validation and enrichment rules, and gives you a clean, governed dataset to work with.

- **[Guide](#guide)**: how each part of the platform works.
- **[Tutorials](tutorials/README.md)**: step-by-step walkthroughs of common tasks.

## What is master data management?

Master data is the core business information that is shared across systems, such as companies, products, suppliers, or locations. When different departments or tools maintain their own copies of this data, inconsistencies creep in. Master data management (MDM) is the practice of creating and maintaining a single, authoritative version of that data.

For a deeper look at the concepts behind the platform, see [Master Data Management Terminology](guide/terminology.md).

## Core concepts

The platform is organised around a few key ideas:

- **Entity types** define the shape of your data: what you're tracking (e.g. "Company", "Product") and which fields each entity has. See [Configuring Entity Types and Fields](guide/entities-and-fields.md).
- **Sources** represent where data comes from. You might have a CRM, an ERP system, and manual entry as three separate sources. Each source contributes its own values, and the platform merges them into a single golden record. See [Working with Sources](guide/sources.md).
- **Sensitivity labels** control who can see what. Fields and entity types are classified as public, internal, sensitive, or secret, and users are granted a corresponding clearance level. See [Working with Sensitivity Labels](guide/sensitivity.md).
- **Survivorship rules** determine which value wins when multiple sources disagree. See [Merging Data Sources](guide/merging.md).

## Typical workflow

A typical setup follows these steps. The **Getting started** checklist on your workspace's home page tracks your progress through the first few.

1. **Define your entity types.** Create entity types for each kind of master data you manage, then add fields with the appropriate data types, validation rules, and sensitivity labels. ([Guide](guide/entities-and-fields.md))
2. **Set up sources.** Create a source for each system that contributes data. Arrange them by priority so the platform knows which system to trust most. ([Guide](guide/sources.md))
3. **Invite users and create API keys.** Add team members with the right roles (viewer, editor, admin) and sensitivity clearances. Create API keys for automated integrations. ([Users](guide/users-and-permissions.md) · [API Keys](guide/api-keys.md))
4. **Load data.** Bring data in through the web UI (manual entry, or pasting or dropping a file on the import page), or push it in through the REST API. ([Manual entry](guide/viewing-and-editing.md) · [Import](guide/importing-data.md) · [API](guide/api.md))
5. **Merge and review.** Values from all sources are consolidated into golden records, automatically or when an admin triggers a merge. A background process continuously checks changed records for incomplete records, potential duplicates, and [soft reference](guide/entities-and-fields.md#soft-references) candidates. Use the data stewardship view to review and resolve these issues. ([Merging](guide/merging.md) · [Data Stewardship](guide/stewardship.md))
6. **Enrich and validate.** Set up API lookups to automatically enrich records from external services, use AI to extract structured attributes from free-text fields, and configure transformations and validation rules to keep data clean. ([Validation and Lookups](guide/validation-and-lookups.md) · [AI Attribute Extraction](guide/ai-extraction.md))
7. **Query your data.** Use natural language to ask questions across your entities. The platform translates your prompt into a validated query using an external AI model. ([Natural Language Query](guide/ai-query.md))
8. **Export and distribute.** Pull consolidated data out through the API, connect Power BI or Fabric to the OData feed, or export it as CSV from the data grid. ([API](guide/api.md) · [OData](guide/odata.md) · [Export](guide/viewing-and-editing.md))
9. **Audit.** Export data change history and privileged action logs as CSV for compliance and auditing. ([Auditing](guide/audit.md))

## Guide

**Concepts**
- [Master Data Management Terminology](guide/terminology.md)

**Setting up**
- [Configuring Entity Types and Fields](guide/entities-and-fields.md)
- [Validation, Transformations, and API Lookups](guide/validation-and-lookups.md)
- [Working with Sources](guide/sources.md)
- [Managing Users and Permissions](guide/users-and-permissions.md)
- [Working with Sensitivity Labels](guide/sensitivity.md)
- [Configuring API Keys](guide/api-keys.md)

**Working with data**
- [Importing Data](guide/importing-data.md)
- [AI Attribute Extraction](guide/ai-extraction.md)
- [Viewing and Editing Data](guide/viewing-and-editing.md)
- [The Data Dictionary](guide/data-dictionary.md)
- [Merging Data Sources](guide/merging.md)
- [Data Stewardship: Fixing Data Errors](guide/stewardship.md)
- [Natural Language Query](guide/ai-query.md)
- [Auditing](guide/audit.md)

**Integrations**
- [The REST API](guide/api.md)
- [The OData Feed (Power BI, Fabric)](guide/odata.md)

## Tutorials

Start with [Your First Golden Record](tutorials/first-golden-record.md). The [tutorials page](tutorials/README.md) lists them all, including API quickstarts in Python, Node.js, PowerShell, C# and curl, Parquet, Power BI and Excel.
