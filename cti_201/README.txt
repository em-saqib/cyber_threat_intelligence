CTI 201 — Scaling BERT for Cyber Threat Intelligence
======================================================

Follow-up to CTI 101. Moves from toy data and single-label setups to
real datasets, multi-label classification, and MITRE ATT&CK mapping.

Prerequisites: work through cti_101 first.

Notebooks
---------

Part 1 — Data acquisition
  01_data_dnrti       Download + parse DNRTI NER corpus (13 entity types).
  02_data_tram        Load TRAM II; build MITRE tactic label space from
                      technique->tactic mapping for multi-label training.
  03_data_aptnotes    Sample ~100 APTnotes PDFs, extract text, save as an
                      unlabeled pool for real-world inference (nb 08).

Part 2 — NER on a real dataset
  04_ner_dnrti_finetune     Re-run the CTI 101 NER recipe on DNRTI.
  05_ner_dnrti_evaluation   Per-entity F1 with seqeval; span-level vs
                            token-level; error-type breakdown.

Part 3 — Multi-label classification + MITRE tactics
  06_multilabel_basics         BCE loss, sigmoid heads, threshold tuning,
                               Hamming loss, micro/macro F1 on a toy set.
  07_mitre_tactics_finetune    Fine-tune on TRAM II tactics; per-tactic F1,
                               co-occurrence, per-tactic threshold tuning.

Part 4 — Wrap-up
  08_pipeline_v2    Unified analyze_report_v2() = DNRTI NER + tactic
                    multi-label classifier with per-tactic thresholds.
                    Plus a BERT vs SecureBERT domain-pretraining A/B.

System dependencies
-------------------
  sudo apt install unrar         # for DNRTI extraction (notebook 01)

Python dependencies
-------------------
  pip install transformers datasets seqeval accelerate torch \
              pdfplumber rarfile scikit-learn requests

Directory layout (data/, processed/, models/ are gitignored)
------------------------------------------------------------
  data/dnrti/      Raw DNRTI archive + extracted BIO files
  data/tram/       Cloned TRAM II repo
  data/aptnotes/   Downloaded PDF sample + extracted text
  processed/       HuggingFace DatasetDicts saved by notebooks 01-03
  models/          Fine-tuned checkpoints

Notes
-----
- Datasets are not redistributed here; each notebook downloads from the
  canonical source on first run.
- APTnotes PDFs are vendor copyrighted — do not redistribute.
- Run notebooks in order. 04-05 depend on 01; 07 depends on 02 and 06;
  08 depends on 04 and 07 (and optionally 03 for the APTnotes demo).
