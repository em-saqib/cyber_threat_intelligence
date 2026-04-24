# CTI 301 — Modern Models for Cyber Threat Intelligence

Follow-up to CTI 101 / 201. Where the earlier series taught BERT-style classification on labeled CTI text, **CTI 301 walks from long-context document classification to a complete dark-web SOC pipeline** — building, in order, the pieces a real dark-web monitoring product is made of.

## Series arc

```
01 ─► 02 ─► 03 ─► 04 ─► 05 ─► 06 ─► 07 ─► 08 ─► 09
 │     │     │     │     │     │     │     │     └── End-to-end SOC pipeline
 │     │     │     │     │     │     │     └────── Credential dump parser (synthetic + HIBP)
 │     │     │     │     │     │     └─────────── Ransomware leak-site detection
 │     │     │     │     │     └────────────────── Asset matching: mock SOC alerting
 │     │     │     │     └──────────────────────── Entity extraction (regex / GLiNER / hybrid + DarkBERT NER)
 │     │     │     └────────────────────────────── DarkBERT fine-tuned on CoDA (9 categories)
 │     │     └──────────────────────────────────── CoDA exploration (dark vs surface)
 │     └────────────────────────────────────────── Agora marketplace classification (ModernBERT)
 └──────────────────────────────────────────────── ModernBERT on full TRAM reports (8k context)
```

## The notebooks

| # | Notebook | Goal | Data | Model |
|---|---|---|---|---|
| 01 | `modernbert_full_reports` | Classify whole CTI reports into MITRE tactics in one forward pass | TRAM II (~150 reports) | ModernBERT-base |
| 02 | `darkweb_listing_classification` | Classify dark-web marketplace listings into CTI-relevant categories | Agora (~16k filtered listings) | ModernBERT-base |
| 03 | `coda_data_exploration` | Understand how dark-web text differs from surface-web — vocabulary, OOV, entities | CoDA + Wikipedia sample | (none — exploratory) |
| 04 | `darkbert_coda` | Fine-tune a domain-pretrained model on dark-web data | CoDA (English, 9 classes) | **DarkBERT** |
| 05 | `entity_extraction` | Extract CTI entities from dark-web pages (CVEs, wallets, emails, actors, malware) | CoDA + silver labels | DarkBERT (token classifier) |
| 06 | `asset_matching` | Match extracted entities against a synthetic customer profile to surface alerts | CoDA + injected hits | (uses nb 05 model) |
| 07 | `ransomware_leak_detection` | Binary classifier for ransomware victim-shaming posts + victim-org extraction | DarkBERT bundled benchmark | DarkBERT (classifier) |
| 08 | `credential_dump_parser` | Parse 4 common dump formats, match employees, enrich with HIBP context | Synthetic dumps + HIBP API | (no ML — pure parsing) |
| 09 | `soc_pipeline_end_to_end` | Chain 04 / 05 / 07 / 08 into one ranked, deduped JSONL alert feed | All of the above | Composes nbs 04, 05, 07 |

## Datasets used

- **TRAM II** — sentence-level CTI labels aggregated to report level. (Open via [center-for-threat-informed-defense/tram](https://github.com/center-for-threat-informed-defense/tram).)
- **MITRE ATT&CK STIX bundle** — for technique → tactic mapping. (Open via [mitre/cti](https://github.com/mitre/cti).)
- **Agora marketplace dump** (2014–2015, ~110k listings, CC0). [Kaggle](https://www.kaggle.com/datasets/philipjames11/dark-net-marketplace-drug-data-agora-20142015).
- **CoDA** — 10k Tor-crawled onion pages, labeled 10 categories. **Gated** access on [Hugging Face](https://huggingface.co/datasets/s2w-ai/CoDA) (research use only, request approval required).
- **DarkBERT bundled benchmarks** — `leaksite_detection_dataset.tsv` (802 posts) and `noteworthy_detection_dataset.tsv` (1873 posts) shipped with the model weights.
- **Wikipedia random sample** — 400 articles for surface-web reference in nb 03.
- **Have I Been Pwned breach catalog** — 975 breaches, public API, no auth.
- **Synthetic credential dumps** — generated at notebook runtime, never real leaked data.

## Models used

- **ModernBERT-base** — `answerdotai/ModernBERT-base`. 8,192-token context.
- **DarkBERT** — `s2w-ai/DarkBERT`. RoBERTa-base continue-pretrained on Tor pages. **Gated** (request from S2W via Hugging Face).
- **GLiNER** — `urchade/gliner_large-v2.5`. Zero-shot NER for fuzzy entity types.

## Setup

```bash
pip install -U transformers datasets scikit-learn torch pandas matplotlib seaborn \
               huggingface_hub kaggle wikipedia gliner seqeval
```

Authentication once:

```bash
huggingface-cli login    # for gated CoDA + DarkBERT access
mkdir -p ~/.kaggle && mv kaggle.json ~/.kaggle/ && chmod 600 ~/.kaggle/kaggle.json
```

## Hardware

Notebook 01 was tuned for CPU (≤8 GB RAM). Notebooks 02–09 expect a CUDA GPU. Tested on an RTX 3070 Laptop (8 GB). Anything with ≥6 GB VRAM should work with fp16; ≥16 GB lets you raise `MAX_LEN` and batch size in nb 02 and 04.

## Run order and dependencies

```
01  ── self-contained
02  ── self-contained (Kaggle download)
03  ── needs CoDA on disk (auto-fetched from HF)
04  ── needs CoDA on disk (auto-fetched)
05  ── needs CoDA on disk
06  ── needs nb 05 model artifact
07  ── needs nb 05 model artifact (for victim extraction); auto-fetches benchmark TSV
08  ── self-contained (synthetic data, HIBP public API)
09  ── needs nb 04, 05, 07 model artifacts + nb 08 synthetic dumps
```

## What this is — and isn't

**This is** a teaching series. Each notebook is intentionally self-contained, with explicit caveats about scope, data quality, and limitations. The pipeline patterns directly transfer to production work.

**This isn't** production. Real dark-web monitoring needs collection infrastructure (Tor scraping, mirror rotation, Telegram/Discord ingestion), MLOps (feedback loops, model retraining), legal/ethics handling specific to your jurisdiction, and operational scaling — none of which is in scope here.

**Synthetic data is used deliberately** for the credential-dump notebook (nb 08) and customer-asset profile (nb 06 / 09). Real stealer logs and real customer data describe real victims; using them in teaching material would be harmful. The parsing and matching code transfers identically to real data.

## Directory layout

```
cti_301/
├── 01_modernbert_full_reports.ipynb
├── 02_darkweb_listing_classification.ipynb
├── 03_coda_data_exploration.ipynb
├── 04_darkbert_coda.ipynb
├── 05_entity_extraction.ipynb
├── 06_asset_matching.ipynb
├── 07_ransomware_leak_detection.ipynb
├── 08_credential_dump_parser.ipynb
├── 09_soc_pipeline_end_to_end.ipynb
└── README.md

# Generated locally on first run, not committed:
data/         # raw downloads (TRAM JSON, Agora CSV, CoDA tar, DarkBERT benchmarks)
processed/    # cached datasets, silver labels, alert JSONL
models/       # fine-tuned checkpoints (multi-GB)
```
