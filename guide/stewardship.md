# Data Stewardship — Fixing Data Errors

When data arrives from external sources, it may be missing the unique key fields that identify a record, it may contain values that fail validation, it may duplicate an existing record, or it may contain text values that need to be matched to records in another entity type. The data stewardship workflow helps you review and resolve these issues.

The stewardship overview surfaces four kinds of problems:

- **Incomplete records** — records missing one or more unique key fields. These cannot be merged into the golden record because the platform does not know which record they belong to.
- **Potential duplicates** — complete records whose merged values are similar enough to warrant review. These may represent the same real-world entity entered from different sources.
- **Reference candidates** — records with [soft reference](entities-and-fields.md#soft-references) fields where the text value has near-matches in the target entity type, but the actual reference has not been set yet.
- **Deferred validation errors** — records that were saved despite failing validation, because [Defer Errors](validation-and-lookups.md#deferred-errors) was enabled for the failing fields. These records contain values that need manual review and correction.

## Background processing

Match candidates (for incomplete records, potential duplicates and reference candidates) are computed in the background rather than when you load the page. Whenever a field value is created, updated or deleted — through the web UI, the API or an import — the affected record is picked up by a background process shortly afterwards, which looks for match candidates.

For fields with vector embeddings enabled, the embeddings are generated first, since matching on those fields depends on them.

## The stewardship overview

Click **Fix data errors** in the sidebar. The link is shown to editors and admins. The overview lists every entity type you have [sensitivity clearance](sensitivity.md) for, with action buttons that show how many issues there are to resolve:

| Button | Opens |
|---|---|
| **Incomplete: Match (n)** | The [incomplete records](#reviewing-incomplete-records) page, for incomplete records that have candidate matches. |
| **Incomplete: Edit (n)** | The data grid, filtered to incomplete records, so that you can [fill in the missing key values](#filling-in-missing-key-values). |
| **Duplicates (n)** | The [potential duplicates](#reviewing-potential-duplicates) page. |
| **References** | The [reference candidates](#reviewing-reference-candidates) page. Shown for every entity type that has soft reference fields; the count is added once there are candidates. |
| **Deferred errors (n)** | The data grid, filtered to [records with deferred validation errors](#fixing-deferred-validation-errors). |

While records of an entity type are still being processed in the background, the row shows badges such as **12 embedding…** or **40 queued…**. These badges disappear once processing has finished.

The page refreshes the counts every 10 seconds, so you do not need to reload it.

> **Screenshot:** The stewardship overview, listing entity types with Match, Edit, Duplicates, References and Deferred errors buttons and a "queued…" processing badge.
> <!-- screenshot: images/stewardship-overview.png -->

## Match scores and the Min. match slider

Both the incomplete records and potential duplicates pages show a **Best Match** column with the similarity score of the closest candidate for each row. Scores are colour-coded:

- **Green (95% and above)** — strong match.
- **Amber (75–94%)** — possible match.
- **Red (below 75%)** — weak match.

Use the **Min. match** slider to filter the list by minimum match score. In its starting position the slider shows **All**, which includes records with no match candidates at all. Moving it raises the minimum score and hides records whose best match falls below it.

The slider also filters the candidates inside an open review tray.

## Match threshold

Each entity type has a **match threshold**, shown as **Min. Match** on the entity type's edit page (admins only). It sets the minimum similarity two records must have to be suggested as a match; the default is 50%. Lower values catch more potential matches but include more false positives. The incomplete records and potential duplicates pages show the configured threshold for the entity type, and the Min. match slider cannot go below it.

The threshold is applied when records are processed in the background. Changing it does not recompute existing suggestions straight away: records are evaluated against the new threshold the next time they change.

## What makes a record incomplete?

A record is considered incomplete when it is missing a value for one or more of the entity type's **unique key fields**, counting the values from all of its sources. For example, if your "Company" entity type has "Company Name" as a unique field, and a CRM integration pushes a record with revenue data but no company name, that record is incomplete.

Incomplete records exist in the system but are not merged into the golden record until they are resolved. There are two ways to resolve one: consolidate it into an existing complete record, or fill in the missing key values.

## Reviewing incomplete records

Click **Match (n)** on the overview. The incomplete records page shows a table of records missing unique key values, along with their field values, sources and best match score. Click **Review** on a row to open the review tray, which shows:

1. **Incoming values** — the values on the incomplete record, along with the source they came from.
2. **Candidate target records** — existing complete records that the platform considers potential matches, ranked by similarity score.

> **Screenshot:** An open review tray on the incomplete records page, with the incoming values at the top, the candidate target records below, and the conflict panel for a selected candidate.
> <!-- screenshot: images/stewardship-incomplete-review.png -->

### Consolidating an incomplete record

Click **Select →** next to a candidate to begin consolidation. The platform compares the incomplete record's values with the candidate's values from the same source, and lists any **conflicts** — fields where both records have a different value. You can expand **Target record values** to see everything the candidate already holds.

For each conflict, choose which value to keep: the incoming value (selected by default) or the existing value on the target. Fields without a conflict are transferred automatically.

Click **Consolidate** to complete the operation. This:

1. Transfers the incoming values to the target record, under the same source.
2. Records the change in the target record's history.
3. Deletes the incomplete record, keeping a pointer to the record it was consolidated into.
4. Marks the target record for re-merging.

If [auto-merge](merging.md#triggering-a-merge) is enabled for the entity type (the default), the golden record is updated automatically shortly afterwards. Otherwise, [run a merge](merging.md#triggering-a-merge) to update it.

### Deleting an incomplete record

If an incomplete record is junk or should not be kept, click **Delete this record** in the review tray and confirm. This deletes the record. The operation cannot be undone through the UI.

### Filling in missing key values

If the missing key values are known, click **Edit (n)** on the overview instead. This opens the data grid filtered to incomplete records, on the source that holds the most of them. The empty unique key cells are highlighted, and you can click them to enter the missing value. The value is saved to the source you are viewing. Once all unique key fields have a value, the record becomes complete and is matched and merged like any other record.

## Reviewing potential duplicates

Potential duplicates are found by comparing the merged values of complete records: text values by spelling similarity (or by meaning, for fields with vector embeddings enabled), numbers and dates by how close they are. Two records whose unique key values differ are never suggested as duplicates; two records whose unique key values are identical are always suggested, with a score of 100%.

Click **Duplicates (n)** on the overview. The potential duplicates page lists records that have been flagged as possible duplicates of other records, ordered by best match score (most likely duplicates first), with their merged values for comparison.

Click **Review** on a row to open the review tray, which shows:

1. The values of the record under review.
2. **Potential duplicate records** — other records with similar values, ranked by similarity score.

> **Screenshot:** An open review tray on the potential duplicates page, with two candidates ticked and the conflict table showing one column per record.
> <!-- screenshot: images/stewardship-duplicates-review.png -->

### Merging duplicate records

Tick the checkbox next to each candidate that is a duplicate of the record under review. You can select several. The platform compares the records' values and shows any **conflicts** in a table with one column per record. For each conflicting field, choose which value to keep; the value of the record under review is selected by default, unless it is empty.

Click **Merge selected** (or **Merge *n* selected**) to complete the operation. The record under review is the one that survives. For each selected duplicate, this:

1. Transfers the duplicate's source values to the surviving record, except for fields where you chose to keep another record's value.
2. Records the change in the surviving record's history.
3. Deletes the duplicate record, keeping a pointer to the record it was merged into.
4. Marks the surviving record for re-merging, so that survivorship rules are applied to the combined values.

Any remaining duplicate suggestions for the surviving record are dismissed at the same time.

### Dismissing duplicates

If a record is not actually a duplicate, click **Dismiss** next to it in the list. The button changes to **Ok?** as a confirmation step — click again to confirm. If you click elsewhere instead, the dismissal is cancelled.

Dismissing a record marks all of its current duplicate suggestions as dismissed. Dismissed suggestions are hidden from the list and do not count towards the duplicate total.

If the records are later updated and become more similar than they were when you dismissed them, the suggestion reappears automatically.

<a id="reference-candidates"></a>

## Reviewing reference candidates

When an entity type has [soft reference](entities-and-fields.md#soft-references) fields, the background process matches their text values against the target entity type. Exact matches are written as reference values automatically. Near matches are kept as candidates for manual review.

Click **References** on the stewardship overview to open the reference candidates page. The page shows a table of unmatched text values, grouped by value — if 15 records all contain the same misspelling, they appear as a single row with a count.

Each row shows:

- **Field** — the soft reference field.
- **Value** — the unmatched text value.
- **Records** — how many records share this value.
- **Suggested Reference** — a dropdown of candidate matches from the target entity type, sorted by match quality (best first). The best match is pre-selected.

For each row, you can:

- **Accept (✓)** — writes the selected reference to all records in the group and removes the row. The reference is written to the source configured as the field's target source for matched references (Manual entry if none is set).
- **Reject (✗)** — dismisses the row without writing any reference. The candidates are removed from the list.

Use the search box to filter by the unmatched text value or the best candidate's label. The list is paginated for large data sets.

> **Screenshot:** The reference candidates page with grouped unmatched values, record counts, the Suggested Reference dropdown and the accept/reject buttons.
> <!-- screenshot: images/stewardship-reference-candidates.png -->

## Fixing deferred validation errors

When data is ingested with [deferred validation errors](validation-and-lookups.md#deferred-errors), the stewardship overview shows the number of affected records per entity type. Click **Deferred errors (n)** to open the data grid filtered to those records.

The grid opens on the source that has the most errors. Cells with validation errors are highlighted in red and are editable — when you are viewing a source other than Manual entry, all other cells remain read-only. To fix an error, click the highlighted cell, enter a valid value and press Enter or Tab. The correction is saved to the source you are viewing (not to Manual entry), and the highlight is removed once the value passes validation.

If a required field was missing entirely, the cell appears empty and highlighted. Enter the missing value to resolve the error.

If errors exist on several sources, use the source dropdown to switch between them.

Values with validation errors are left out when the golden record is calculated, so the merged value is updated once the error is fixed.

## Preventing incomplete records

Incomplete records are typically caused by external sources that do not provide all the unique key fields. To reduce them:

- Make sure API integrations always include the unique key fields when pushing data.
- Set up [API lookups](validation-and-lookups.md#api-lookups) to enrich records with missing key data from external services.

---

[← Back to overview](../README.md)
