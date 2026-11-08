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
| `pubmed_results_19000101_20261031_00-03-23.csv` | 22,292 (4,983 without abstract) | PubMed fetch, 1900/01/01–2026/10/31 (`[Date - Create]`), fetched 5 Nov 2026 with `SEARCH_STRING_WITHOUT_STUDY_TYPE` from `get_pubmed_data.py`.| `studies_20261031_00-03-00.csv` | 22,292 | The PubMed fetch above with the prediction of the **new relevance model** (`biolinkbert_relevant_20261107`; `prediction`, `probability`). 6,337 relevant, of which 259 have no abstract (classified on the title only, not loaded into the database). Ubelix job 21507639, 8 Nov 2026. |
| `studies_relevant_deduplicated_excluded_20261031_00-00-00.csv` | 3,464 | **Automated route seed file.** The relevant records with an abstract (6,078), minus records matching an ASReview-excluded or after-stopping record (322) or an ASReview-included record (2,292), matched on normalised PMID **or** DOI. Written by `analysis/analysis_pubmed_new_post_revision.ipynb`. In `init_db.sh` this creates the automated papers (`-s`). |
| `class_predictions_20240101_00-06-48_thresholded.csv` | 50,118 (2,352 studies) | New models, 17 classification tasks, on `as_review_studies_relevant_unannotated_…`. Multi-label tasks at the tuned thresholds of `pipeline/model_paths.json` (`probability >= threshold`); single-label = argmax. |
| `ner_predictions_20240101_00-00-27.csv` | 2,352 | New NER model (`biomedbert-abstract_ner_bio_20261106`, selected on dosage) on the same studies. Same file name as the old file in `data/history/`. |
| `class_predictions_20261031_00-16-34_dedup_thresholded.csv` | 74,585 (3,464 studies) | New models on the PubMed relevant set, restricted to the ids of the deduplicated seed file, thresholds as above. Restricting is needed: `populate.py` falls back to a `pubmed_id` lookup, so predictions of removed duplicates would otherwise attach to the matching manual paper. |
| `ner_predictions_20261031_00-01-01_dedup.csv` | 3,464 | New NER model on the PubMed relevant set, restricted to the deduplicated ids. |

The unthresholded prediction files (`--keep-all-labels`, all labels with probabilities) and the unfiltered NER file of the PubMed set are in `psynamic_predictions.tar.gz` (research storage), not here.
