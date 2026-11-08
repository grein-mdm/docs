# AI Attribute Extraction

Attribute extraction uses an external AI model to pull structured values out of free text. For example, it can split an address string into separate street, postal code and city fields, or read a product description and fill in its colour and material. You describe what to extract in plain language, check the results on a small sample, and then run the extraction across all matching records. The extracted values are written to a source of your choice and merged into the golden record like any other source data.

## Before you start

Three things need to be in place before you can use attribute extraction:

- **The feature must be included in your plan.** Attribute extraction is part of the external AI features, which also include [natural language query](ai-query.md). If they aren't included in your plan, **Attribute extraction** appears in the sidebar with a lock icon.
- **An AI model must be connected.** A workspace admin has to connect an external AI model under **Settings > External AI Model**, with a provider, an API key and a model. See [Natural Language Query: Setup](ai-query.md#setup) for the supported providers and settings. Attribute extraction uses the same connection as natural language query. Until a model is connected, the extraction page only shows a link to Settings.
- **You need the editor or admin role.** Viewers can't open the extraction page. See [Roles](users-and-permissions.md#roles).

It is also a good idea to [create a dedicated source](sources.md#creating-a-source) for extracted values, such as "AI extraction", before your first run. This keeps AI-generated values separate from data you have imported or entered by hand. See [Where extracted values go](#where-extracted-values-go).

> [!WARNING]
> When you preview or run an extraction, the text values being processed are sent to the external AI provider your admin has configured. Before you proceed, make sure your data classification and privacy policies allow this data to be sent to that provider.

## Setting up an extraction

Click **Attribute extraction** under **Tools** in the sidebar, then choose an **Entity type**. The rest of the form appears:

| Setting | Description |
|---|---|
| **Saved configuration** | Choose a configuration you have run before to fill in the form, or leave it at **New configuration…** to start from scratch. See [Saved configurations](#saved-configurations). |
| **Input source** | The [source](sources.md) whose values you want to read. |
| **Input field** | The field that contains the free text. Only `text`, `enum` and custom-type fields can be used as input. |
| **Output source** | The source that the extracted values are written to. |
| **Output fields** | One or more fields to fill with extracted values. The input field can't also be an output field. |
| **Extraction instructions** | A plain-language description of what to extract, for example *"Extract street name, street number, postal code and city from the address"*. |

The AI model receives the display name, machine name and field type of each output field together with your instructions, so clear and descriptive field names help it put each value in the right place.

> **Screenshot:** The attribute extraction page with an entity type selected, an address field chosen as input and street, postal code and city ticked as output fields.
> <!-- screenshot: images/ai-extraction-setup.png -->

## Previewing the results

Click **Preview** to try your instructions on a small sample. Up to five values that haven't been processed yet are picked at random from the input field and sent to the AI model. The results are shown in a table, with the original input in the first column and one column per extracted attribute. The number of tokens used is shown above the table.

Previewing doesn't save anything. If the results aren't what you expect, adjust your instructions or output fields and preview again. Every preview is a call to your AI provider and uses tokens.

The **Run extraction** button stays disabled until a preview has succeeded. If you then change the input field or choose a different saved configuration, you have to preview again before you can run.

> **Screenshot:** The preview table showing five address strings next to the street, postal code and city values the AI model extracted from them.
> <!-- screenshot: images/ai-extraction-preview.png -->

## Running an extraction

Click **Run extraction** to process every eligible value. A value is eligible if it:

- belongs to the selected input field and input source,
- is not empty,
- belongs to a record that hasn't been deleted, and
- hasn't been processed by an earlier extraction run.

Values are sent to the AI model in small batches. While the job runs, a progress panel shows the number of rows processed out of the total, the number of tokens used, and the number of rows that failed. Processing continues in the background, so you can leave the page. Note that the progress panel is only shown on the page where you started the run. It doesn't reappear if you open the extraction page again later.

Click **Stop** to cancel the run. Values that have already been processed keep their results, and the remaining values are picked up the next time you run the same extraction. The same applies if a run is interrupted for any other reason.

When the run ends, its status changes to one of the following:

| Status | Meaning |
|---|---|
| **Completed** | All eligible values have been processed. |
| **Stopped** | You stopped the run before it finished. |
| **Error** | The AI provider returned an error, for example because of an invalid API key or an exceeded quota. The run stops, and the unprocessed values are picked up by the next run. |

If the AI model returns no usable result for an individual value, that value is retried up to three times. If it still fails, it is counted under **Failed** and stays eligible, so a later run will try it again.

> **Screenshot:** The extraction progress panel part-way through a run, with the progress bar, row count, token count and Stop button.
> <!-- screenshot: images/ai-extraction-progress.png -->

## Where extracted values go

Extracted values are written to the selected output fields in the **output source**, on the same record that the input value came from. Extracted values go through the output fields' [transformations](validation-and-lookups.md#transformations) and [validation rules](validation-and-lookups.md#validation-rules) first. A value that fails validation is skipped and the existing value is kept. Otherwise, whenever the AI model returns a value for a field, it replaces that field's existing value in the output source. If the model can't determine a value (it returns nothing for that field), any existing value in the output source is left as it is.

Because extracted values are ordinary source data, they become part of the golden record through [merging](merging.md), following each field's [survivorship rule](merging.md#survivorship-rules). Use source priority to decide how much weight AI-generated values get. For example, place your extraction source below sources that hold verified data, so that extracted values only fill gaps. The changes appear in each record's [history](viewing-and-editing.md#record-history), attributed to the user who started the run.

> [!WARNING]
> The **Output source** list includes every source in the workspace, including **Manual**. If you choose a source that already holds real data, extracted values will overwrite it, and you won't be able to tell AI-generated values apart from the original data afterwards. Use a dedicated source for extraction output.

## Re-processing values

Each value is processed only once. Later runs of the same extraction skip values that have already been processed, so running it again only handles new or unprocessed values.

To process everything again, for example after improving your instructions, select the same entity type, input source and input field, and click **Reset processed rows**. The page tells you how many rows were reset, and those rows will be processed again on the next run.

## Saved configurations

When you run an extraction, its settings (input source, input field, output source, output fields and instructions) are saved automatically. Each combination of entity type, input source and input field holds one saved configuration, so running the same combination again with different settings replaces the saved one. A configuration is only saved if the run found at least one value to process.

To reuse a configuration, pick it from the **Saved configuration** list, which shows the input field and the start of the instructions. Then preview and run as usual.

## Limitations and things to keep in mind

- **Data leaves the platform.** Every preview and run sends the input values to the external AI provider configured by your admin. You can only choose entity types and fields at or below your own [sensitivity level](sensitivity.md), but anything you can see can be sent, so check your policies before extracting from fields that hold personal or confidential information.
- **Results are AI-generated.** The model can misread, guess or leave out values. Always preview first, and check a sample of records in the output source after a run.
- **Each extracted attribute is a single value.** For every output field, the AI model returns a single text, number or true/false value, or nothing. Simple field types such as `text`, `number`, `date` and `boolean` work best.
- **Token usage costs money.** Previews and runs use tokens on your AI provider account, and any charges are billed by the provider. The token counts on the page help you estimate the cost. Rate limits set under **Settings > External AI Model** apply.
- **Runs aren't automatic.** New records or values aren't extracted automatically. Run the extraction again to process values added since the last run.
- **The original language is kept.** The AI model is instructed not to translate the input, so extracted values stay in the language of the source text.

---

[← Back to overview](../README.md)
