# Master Data Management Terminology

This page explains the key terms used throughout Grein and this documentation.

## Entity type

An entity type is a category of master data — for example, "Company", "Product", or "Supplier". Each entity type has its own set of fields that describe the attributes you want to track. Entity types are roughly equivalent to tables in a relational database.

Every entity type has a **display name** (shown in the UI) and a **slug** (a lowercase API name used in API URLs and the OData feed). See [Configuring Entity Types and Fields](entities-and-fields.md).

## Field

A field is a single attribute on an entity type — for example, "Company Name", "Founded Date", or "Revenue". Each field has a [field type](entities-and-fields.md#field-types) (text, number, date, boolean, amount, and so on), and can be marked as required or unique.

Fields also carry a [sensitivity label](sensitivity.md) and a [survivorship rule](merging.md#survivorship-rules) that controls how values from different sources are merged.

## Record

A record is a single instance of an entity type — one company, one product, one supplier. Each record has an ID and a set of field values. A record can have values from multiple sources, and those values are merged into a single golden record.

## Source

A source represents where data comes from. Common examples include CRM systems, ERP exports, manual data entry, or API enrichment services. Each source contributes its own set of values for a record, and Grein merges them based on source priority and survivorship rules.

Two sources are built in: **Manual entry** (for data entered through the web UI) and **Merged results** (the consolidated result). See [Working with Sources](sources.md).

## Golden record

The golden record is the merged, authoritative version of a record — the single source of truth produced by combining values from all sources according to survivorship rules. In Grein, this is what you see when you view the **Merged results** source.

## Survivorship rule

A survivorship rule determines which value wins when multiple sources provide different values for the same field. The options are source priority (trust the highest-ranked source), most recent, consensus, average, and median. See [Merging Data Sources](merging.md).

## Sensitivity label

A sensitivity label classifies how restricted a piece of data is. Grein uses four levels: **public**, **internal**, **sensitive**, and **secret**. Labels are assigned to entity types, fields, users, and API keys. See [Working with Sensitivity Labels](sensitivity.md).

## Unique key

A unique key is a field (or combination of fields) that uniquely identifies a record within an entity type. When data is imported or pushed via the API, Grein uses the unique fields to match incoming rows to existing records, enabling upsert (update-or-insert) behaviour. Values from different sources that share the same unique key are merged into the same golden record.

## Soft reference

A soft reference is a text field that is linked to a text field on another entity type for cross-entity fuzzy matching. Unlike a regular reference field (which stores an explicit record ID), a soft reference works by comparing text values — through vector similarity when embeddings are enabled, otherwise through text similarity — and suggests which record the value refers to. See [Soft references](entities-and-fields.md#soft-references).

## Vector embedding

A vector embedding is a numeric representation of a text value that captures its meaning. When vector embeddings are enabled on a text field, each value is converted into a vector in the background, which is then used for similarity-based matching in duplicate detection and soft reference resolution. This allows Grein to match values that are similar but not identical, such as misspellings, abbreviations, or name variations.

## Data stewardship

Data stewardship is the process of reviewing and correcting data quality issues. In Grein, this includes reviewing incomplete records (records missing one or more unique key fields), resolving potential duplicates (merged records that appear to represent the same entity), reviewing [reference candidates](stewardship.md#reference-candidates) (soft reference matches that need manual confirmation), and fixing deferred validation errors. See [Data Stewardship](stewardship.md).

## Transformation

A transformation is a processing step applied to incoming field values before validation. Examples include trimming whitespace, converting to uppercase, or extracting the first word. Transformations help normalise data from inconsistent sources. See [Validation, Transformations, and API Lookups](validation-and-lookups.md).

## API lookup

An API lookup is an automated enrichment rule that calls an external web service when a record is saved, then maps the response back into the record's fields. For example, you could look up a company's timezone from its country code. See [Validation, Transformations, and API Lookups](validation-and-lookups.md#api-lookups).

## Background processing

Some work happens in the background rather than while you wait: generating vector embeddings for text fields, computing match candidates (for incomplete records, potential duplicates, and soft reference candidates), merging records into golden records for entity types with auto-merge enabled, and applying retention policies. Background processing picks up changes continuously, so results usually appear shortly after data is saved. A large import may take a little longer to work through.

## Tenant

A tenant (also called a workspace) is an isolated environment. Each tenant has its own entity types, records, sources, users, and API keys, and data is kept fully separate between tenants. Users can belong to multiple tenants with different roles in each.

---

[← Back to overview](../README.md)
