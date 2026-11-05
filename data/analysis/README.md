# Seed and analysis data

Files used to build the initial database ([pipeline/init_db.sh](../../pipeline/init_db.sh)) and for the manuscript analysis ([analysis/analysis_pubmed_new.ipynb](../../analysis/analysis_pubmed_new.ipynb)).

Rows are records. Studies are linked across files by `id`. ASReview records use their ASReview `id`, and PubMed records use the PMID.

## Created manually

| File | Rows | What it is |
|---|---|---|
| `as_review_studies_relevant_with_info_20240101_00-00-00.csv` | 3,313 | All studies screened as **relevant** in ASReview (`included=1`), with metadata. `pubmed_id` was added afterwards by looking up each title on PubMed (`get_pubmed_id_from_title`); 917 records still have none. In `init_db.sh` this creates the manual papers (`-s`). |
| `as_review_studies_relevant_unannotated_20240101_00-00-00.csv` | 2,352 | The relevant studies above **minus the 961 annotated ones** (`id`, `text` only). Input for predicting the 17 classification tasks and NER on the manually included studies. |
| `as_review_studies_excluded_with_pubmed_id.csv` | 2,370 | Studies screened and **excluded** in ASReview (`included=0`). `pubmed_id` added by title lookup (957 found). Used to remove these studies from the automated route. |
| `as_review_studies_excluded_after_stopping_with_pubmed_id.csv` | 3,939 | Records **never screened** because the ASReview stopping rule was reached (`included` empty). `pubmed_id` added by title lookup (1,520 found). Also removed from the automated route. |
| `class_predictions_manual_20250127_00-00-00.csv` | 18,717 (961 studies) | **Human labels** of the 961 annotated studies for the 17 classification tasks, in prediction format (`probability=1.0`, `model=manual_annotation`). Covers all train, val and test IDs of `training_round2/<task>/`. |
| `ner_bio_966_manual.jsonl` | 966 | **Human dosage NER annotations** in BIO format (`tokens`, `ner_tags`, `id`). These are the 961 annotated studies plus 5 that were rejected as irrelevant in the classification review but accepted in the NER review (4670, 4953, 2675, 5936, 7164). Those 5 aren't in the database, so their tags aren't loaded. |

## Created automatically

| File | Rows | What it is |
|---|---|---|
| `pubmed_results_19000101_20261031_00-03-23.csv` | 22,292 (4,983 without abstract) | PubMed fetch, 1900/01/01–2026/10/31 (`[Date - Create]`), fetched 5 Nov 2026 with `SEARCH_STRING_WITHOUT_STUDY_TYPE` from `get_pubmed_data.py`.