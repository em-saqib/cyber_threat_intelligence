Repo 401: Dark-Web Collection & Monitoring — Starter
=====================================================

Follow-up to CTI 301. Builds the small but honest end-to-end loop that 301
deliberately skipped: collect -> store -> classify -> monitor.

Scope: starter-level. Single-threaded crawls, public sources only, no
training (we reuse the DarkBERT classifier from cti_301/nb 04). Designed
to run on a laptop with 1x 8 GB GPU.

Prerequisites: cti_101, cti_201, cti_301. The classifier from cti_301
notebook 04 must be on disk under cti_301/models/.

Notebooks
---------

  01_tor_setup_and_safe_fetch        Install Tor; fetch onion pages via
                                     SOCKS; circuit rotation; rate-limit
                                     and ethics checklist.

  02_seed_and_crawl_small            Seed from Ahmia; crawl ~100 pages
                                     synchronously; save raw HTML +
                                     metadata to data/raw_html/.

  03_parse_and_store                 Clean text with trafilatura; near-dup
                                     filter with MinHash; persist to
                                     DuckDB at db/cti.duckdb.

  04_classify_with_darkbert          Load cti_301 DarkBERT checkpoint;
                                     batch fp16 inference over the
                                     corpus; write predictions back to
                                     DuckDB.

  05_telegram_ingestion_basic        Pull recent messages from 2-3 public
                                     CTI/leak Telegram channels with
                                     Telethon; same parse path as nb 03.

  06_monitoring_loop_and_alerts      Scheduled re-crawl; diff against
                                     DuckDB; emit alerts for new
                                     high-severity hits; tiny Streamlit
                                     view.

System dependencies
-------------------
  sudo apt install tor

Python dependencies
-------------------
  pip install stem requests[socks] pysocks trafilatura datasketch \
              duckdb telethon transformers torch streamlit

Hardware
--------
CPU is sufficient for notebooks 01-03, 05, 06. GPU is used only in
notebook 04 (DarkBERT fp16 inference at batch size 16 needs ~3 GB VRAM).
No fine-tuning is performed in this repo.

Run order
---------
Linear, 01 -> 06. Each notebook reads artifacts produced by the previous
one (HTML files in data/, rows in db/cti.duckdb).

  01 ── self-contained (just sets up Tor)
  02 ── needs Tor running from 01
  03 ── needs HTML produced by 02
  04 ── needs DuckDB from 03 + cti_301/models/darkbert_coda
  05 ── self-contained (Telegram API), writes to same DuckDB
  06 ── needs DuckDB populated by 03/04/05

Directory layout
----------------
cti_401/
├── 01_tor_setup_and_safe_fetch.ipynb
├── 02_seed_and_crawl_small.ipynb
├── 03_parse_and_store.ipynb
├── 04_classify_with_darkbert.ipynb
├── 05_telegram_ingestion_basic.ipynb
├── 06_monitoring_loop_and_alerts.ipynb
└── README.txt

  # generated locally on first run, not committed:
  data/raw_html/        nb 02 output (one .html per fetched page)
  data/telegram/        nb 05 output (raw message JSONL)
  db/cti.duckdb         nb 03 onward (corpus + predictions + alerts)
  alerts/               nb 06 output (alert JSONL feed)

Scope guardrails
----------------
- Allowlist categories only. Do not crawl or classify CSAM-adjacent
  content. The seed list in nb 02 explicitly filters by Ahmia category.
- Public Telegram channels only. No private/invite-only groups.
- OPSEC: run inside a dedicated VM. Do not point the asset-matching
  step (carried over from cti_301 nb 06) at real customer data.
- Legal: dark-web collection is jurisdiction-dependent. Confirm what
  is permitted where you operate before running nb 02.
- Rate limits: nb 02 sleeps between requests. Do not remove this.

What this is - and isn't
------------------------
This is a teaching starter for collection + monitoring. The patterns
(Tor SOCKS, dedup, DuckDB-as-substrate, diff-based alerting) transfer
to production work but the implementation here is intentionally minimal:
synchronous, single-source, no captcha handling, no mirror rotation,
no authenticated forum access, no actor re-identification.

Those topics are deferred to a future repo.

Notes
-----
- Tor circuits and onion availability are unstable. Re-run nb 02 if a
  fetch fails; the crawler is idempotent (skips URLs already in data/).
- Telegram channel IDs change. The list in nb 05 is a starting point;
  replace with channels you have authorization to monitor.
- DarkBERT weights are gated on Hugging Face; you must already have
  access from cti_301.
