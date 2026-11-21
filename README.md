# BOLT: Browser-Only Labelling Tool
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.24420609.svg)](https://doi.org/10.5281/zenodo.24420609)

BOLT is a single-file browser application for expert review of annotations extracted from free text, such as classifications produced by a large language model (LLM). It shows each provisional annotation next to its source document and highlights the evidence the model cited. Reviewers can confirm, correct or reject each annotation and add annotations the model missed. The exported CSV keeps the original model output alongside every review decision, so you can trace each revision and analyse extraction errors.

BOLT runs entirely in the browser. It needs no installation, server, database or user account, and imported data stay on your computer.

## Requirements

- A modern desktop browser. BOLT was tested in Google Chrome.
- Optional periodic autosave needs the File System Access API, which Chromium-based browsers (Chrome, Edge) support. In other browsers, save your work with manual CSV export.

## Installation

No installation or build step is needed.

1. Download [`src/BOLT.html`](./src/BOLT.html).
2. Open the downloaded file in your browser.

## Inputs and outputs

| Input | Format | Required |
|---|---|---|
| Source records | CSV with a unique ID column and a text column | Yes |
| Annotation schema | JSON object mapping each variable to its permissible classifications | Yes |
| Provisional annotations | CSV with columns for the record ID, evidence, variable and classification, plus optional reason and uncertainty columns. You map your column names in BOLT, so no fixed names are required. | No |

**Output:** a CSV file (ending in `_marked.csv`) with one row per annotation. Each row contains the original classification, evidence and reason, the reviewer's verdict, any revised values, and flags for new and rejected annotations.

## Repository structure

```
BOLT/
├── README.md
├── LICENSE.txt
├── CITATION.cff
├── src/
│   └── BOLT.html                     # the application
└── examples/
    ├── synthetic_source_notes.csv    # three fictional discharge summaries
    ├── bolt_schema.txt               # annotation schema (JSON)
    ├── annotations.csv               # provisional annotations to review
    └── gpt-5.6-terra_results.csv     # raw upstream model output (reference only)
```

## Illustrative example

Use BOLT to review nine provisional annotations across three fictional discharge summaries. Each note has one annotation each for housing insecurity, caregiver presence and caregiver stress. The examples contain synthetic patient narratives, and the supplied model predictions are starting points for expert review. You can complete the exercise locally with a browser and the files below.

| File | Role |
|---|---|
| [src/BOLT.html](./src/BOLT.html) | Application. Download the file and open the local copy in a browser. |
| [examples/synthetic_source_notes.csv](./examples/synthetic_source_notes.csv) | Three source notes identified by `SYN-001`, `SYN-002` and `SYN-003`. |
| [examples/bolt_schema.txt](./examples/bolt_schema.txt) | JSON text defining the three variables and their permissible labels (`Present`, `Negated`, `NA`). |
| [examples/annotations.csv](./examples/annotations.csv) | Review input, with one row per provisional annotation, including the evidence, reason and model-reported uncertainty. |

The additional file [examples/gpt-5.6-terra_results.csv](./examples/gpt-5.6-terra_results.csv) contains the upstream model results, with JSON stored in CSV cells. It is included for reference. Use `annotations.csv` for the BOLT import. The worked example requires no model account, API key or new extraction run.

### Set up the review

1. Open the local copy of `BOLT.html`. In **Configure Schema**, paste the entire contents of `bolt_schema.txt` into **Schema JSON**. Press **Tab** to apply the schema. Check that the visual builder shows three variables, then select **Next**.
2. In **Upload Data CSV**, choose `synthetic_source_notes.csv`. Once the file has loaded, select **Next**.
3. In **Select ID & Text Column**, select `unique_note_ID` for the ID and `note_text` for the text. Select **Next**.
4. In **Upload Annotations CSV**, choose `annotations.csv`. Set the **UID column** to `Index`. Its values match the source note IDs even though the column names differ. Check the remaining mappings below and select **Next**.
5. In **BOLT Behaviour & Preferences**, leave **Find All Instances** unchecked. Manual saving is sufficient for this exercise. Select **Start Verification**.

| BOLT annotation field | Column in annotations.csv |
|---|---|
| UID | `Index` |
| Evidence / text | `evidence` |
| Variable | `variable` |
| Classification | `classification` |
| Uncertainty | `uncertainty` |
| Reason | `reason` |

**Check your setup.** A fresh review opens on `SYN-001` and shows **3 total rows**, **9 total annotations**, **9 pending** and **0% progress**. If you see notes but no cards, use the gear icon to reopen setup and check the UID mapping in step 4.

### Try two review decisions

The left panel tracks progress, the centre displays the note, and the right panel contains the annotation cards. Click a card to locate its evidence when a match is available.

1. **Confirm a supported classification.** Navigate to `SYN-003` using **Next** or jump to row 3. Select the **Caregiver stress** card. The highlighted passage says the partner "denied feeling strained by these tasks". This explicitly supports the proposed classification of `Negated`. Select **Correct**.
2. **Correct an unsupported inference.** On the same note, examine **Housing insecurity**. The model proposed `Negated` based on the patient living with a partner and returning home. For this exercise, reserve `Negated` for an explicit denial of housing insecurity and use `NA` when the note does not establish housing security. Select **Wrong**, change **Fix Classification** to `NA`, and enter a reason such as "The note describes where the patient lives but does not address housing insecurity." Select **Save Changes**.

After these two actions from a fresh session, the sidebar should show **1 verified correct**, **1 marked wrong**, **7 pending** and **22% progress**. The corrected annotation counts as *marked wrong* because the verdict applies to the original prediction.

**Interpreting NOT FOUND.** BOLT looks for a continuous passage within the note. Five supplied cards display this flag across the three records. Four evidence fields combine excerpts from different locations, and the `SYN-002` caregiver-stress annotation has an empty evidence field with a classification of `NA`. The flag reports the result of text matching. Read the note before deciding whether the classification is supported.

To explore further, use **Wrong** to select replacement evidence directly from the note or **Wrong Altogether** to reject an annotation. To add an annotation the model missed, highlight a passage in the main note, choose its variable and classification, and save it.

### Save and resume

Select **save now** to export your review to CSV. With autosave disabled, the browser downloads a file ending in `_marked.csv`. Each row links an annotation to its source note. After the worked correction, the row for `SYN-003` and `housing_insecurity` should contain `Negated` in `original_classification`, `wrong` in `verdict` and `NA` in `fixed_classification`. Pending annotations are also included, so you can save before finishing the dataset.

To continue later, open BOLT again, supply the same schema and source records, and select **restore previous progress** in the annotation upload step. Choose the saved `_marked.csv` file. Optional periodic autosave is available in browsers that support writing to a user-selected local file.

**Export detail for this version.** Each exported data row has two trailing span-position fields beyond the 15 named columns. If your CSV reader requires equal header and row lengths, read the first 15 fields by position for analysis. BOLT's restore function reads the named review fields and does not restore character offsets.

## Citation

If you use BOLT, please cite the software and the accompanying article. Citation details are in [`CITATION.cff`](./CITATION.cff), and GitHub shows them under **Cite this repository**.

## Licence

BOLT is released under the [MIT License](./LICENSE.txt).

## Contact

For questions or bug reports, open an issue on GitHub or email [liu_yanzhao@outlook.com](mailto:liu_yanzhao@outlook.com).