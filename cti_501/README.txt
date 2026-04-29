Repo 501: Agentic AI for Cyber Threat Intelligence — Starter
=============================================================

Follow-up to CTI 401. Builds an agentic-AI loop on top of the
collection + classification + alerting substrate that 401 produced.

Scope: starter-level. Local LLM (Ollama + Llama 3.1 8B), single
laptop, 1x 8 GB GPU. No paid APIs. No async or multiprocess. The
agent is small, the tools are small, the failures are visible —
which is the lesson.

Prerequisites
-------------
- cti_401 has been run end-to-end (db/cti.duckdb populated).
- Ollama installed and running (`ollama serve`).
- Model pulled: `ollama pull llama3.1:8b`
- pip install: ollama nbformat duckdb requests

Notebooks
---------

  01_tools_not_agents              Wrap 4 read-only tools over the
                                   cti_401 DuckDB substrate. Build
                                   a tool registry + dispatcher.
                                   No LLM in this notebook.

  02_single_step_tool_use          Connect Llama 3.1 to the registry.
                                   Question -> tool call -> tool
                                   result -> answer. One round-trip.
                                   Full message-level visibility.

  03_planning_loop                 Wrap nb 02 in a think -> act ->
                                   observe loop with hard caps:
                                   MAX_STEPS=4, MAX_TOOL_CALLS=6,
                                   tool-output truncation at 1500
                                   chars. Why caps exist: small
                                   models can loop forever.

  04_memory                        Two new tools (remember_note,
                                   recall_notes) backed by an
                                   agent_memory table in DuckDB.
                                   Multi-turn loop with rolling
                                   summary so context stays small.

  05_two_agents_sequential         Collector + Analyst, each with
                                   its own tool subset. They talk
                                   via an agent_tasks queue in
                                   DuckDB, not via shared messages.
                                   Sequential, not concurrent.

  06_eval_and_guardrails           Trace every model call to JSONL.
                                   Replay without re-spending
                                   compute. Count failure modes
                                   (cap_hit, repeated_call, bad_args,
                                   unknown_tool). Guardrails: tool
                                   allowlist, per-run budget, human-
                                   in-the-loop confirmation gate
                                   for mutating tools.

System dependencies
-------------------
  Ollama: https://ollama.com (curl install or your package manager)

Python dependencies
-------------------
  pip install ollama nbformat duckdb requests

  (DuckDB and requests are already in your env from cti_401.)

Hardware
--------
The agent itself is hosted by Ollama on your local GPU. Llama 3.1 8B
at the default quantization needs ~5 GB VRAM. On an 8 GB GPU this
fits comfortably; do not also load DarkBERT in the same kernel.

Run order
---------
Linear, 01 -> 06. Each notebook re-imports nb 01 (and sometimes nb
04) so they can be opened independently as long as the underlying
DuckDB exists.

  01 -- self-contained (no LLM)
  02 -- needs nb 01 + Ollama
  03 -- needs nb 01 + Ollama
  04 -- needs nb 01 + Ollama; also writes to DuckDB
  05 -- needs nb 01 + nb 04; runs both agents end-to-end
  06 -- needs nb 01 + nb 04; produces traces/agent_traces.jsonl

Directory layout
----------------
cti_501/
|- 01_tools_not_agents.ipynb
|- 02_single_step_tool_use.ipynb
|- 03_planning_loop.ipynb
|- 04_memory.ipynb
|- 05_two_agents_sequential.ipynb
|- 06_eval_and_guardrails.ipynb
|- README.txt

  # generated locally on first run, not committed:
  traces/                  nb 06 output (per-call JSONL)

  # external substrate (read by all notebooks, not part of this repo):
  ../cti_401/db/cti.duckdb (created by cti_401)

Scope guardrails
----------------
- Read-only DuckDB connection in nb 01-03. Mutations only in nb
  04-06, and only through tools the model can be denied.
- Tool outputs are truncated to 1500 chars before being fed back
  to the model. This is the single most important detail for
  keeping a local 8B model on track.
- No fine-tuning. No re-training. The agent is the model
  Ollama hosts; you are orchestrating it.

What this is - and isn't
------------------------
This is a teaching starter for agentic AI applied to CTI work. The
patterns (schemas, dispatcher, planning loop, role isolation, trace
+ replay) transfer to production work but the implementation here
is intentionally minimal: synchronous, single-process, local model,
no scheduler.

Notes
-----
- Local 8B tool-use is real but imperfect. Expect a noticeable
  rate of malformed JSON args, repeated calls, and "wrong tool"
  decisions. nb 06 catalogues them. This is a feature of the
  teaching, not a bug.
- DuckDB allows only one writer at a time. If a notebook errors
  on connect, shut down the kernel of any other open notebook
  (cti_401 nb 04/06 is the usual culprit) and try again.
- Not all questions have answers in the corpus. The stress-test
  questions in nb 03 and nb 06 are deliberately unanswerable to
  show how the agent handles "I cannot do this."
