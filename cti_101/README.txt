Cyber Threat Intelligence - A Beginner Guide
==================================================

A hands-on learning series (11 notebooks) that builds from BERT fundamentals
to a deployable NER + classification pipeline for CTI reports.

Notebooks
---------

Part 1 — BERT foundations
  01_tokenization_basics            How BERT breaks CTI text into tokens; why
                                    subword fragmentation matters.
  02_bert_outputs_cls_vs_tokens     Which BERT outputs power classification
                                    (CLS) vs NER (per-token vectors).

Part 2 — Named Entity Recognition
  03_bio_tagging_for_ner            The BIO labeling scheme for marking
                                    entity boundaries.
  04_subword_label_alignment        Mapping word-level BIO labels onto
                                    BERT's subword tokens via word_ids().
  05_ner_finetuning_end_to_end      Full NER fine-tune: data prep -> train
                                    -> inference.
  06_ner_evaluation_with_seqeval    Token-level vs span-level metrics,
                                    per-entity F1, error patterns.

Part 3 — Report Classification
  07_classification_data_prep       CTI category schema + stratified splits.
  08_classification_finetuning      Train BertForSequenceClassification;
                                    per-class evaluation.

Part 4 — Production concerns
  09_handling_long_reports          Chunking inputs >512 tokens with overlap
                                    and aggregating predictions.
  10_domain_models_securebert       Swap vanilla BERT for SecureBERT
                                    (cyber-pretrained) and compare.
  11_inference_pipeline             Unified analyze_report() combining NER
                                    + classification.

Artifacts in this directory
---------------------------
  ner-cti-bert / ner-cti-bert-final   Checkpoints + final NER model (05, 06)
  cls-cti-bert / cls-cti-bert-final   Checkpoints + final classifier (08)
  cls-data/                           Classification dataset splits (07)

Setup
-----
  pip install transformers datasets seqeval accelerate torch

Recommended order: run notebooks 01 -> 11 in sequence. Notebooks 05 and 08
train models consumed by later notebooks (09, 11), so do not skip them.
