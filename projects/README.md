# Portfolio projects — ship 4–5 artifacts, not 100 notebooks

> The single highest-leverage thing you can do to land an AI Product Engineer role. Updated **2026-07**.

[⬅ Roadmap index](../README.md) · [Are you ready to apply? →](../readiness-check.md)

---

Employers don't hire on notebooks — they hire on **evidence that you shipped a measured probabilistic system**. So the rule for this repo is blunt: **ship 4–5 real projects, not a hundred half-finished notebooks.** Each project below is deliberately shaped so that finishing it produces one *proof artifact* a hiring manager can click.

**The non-negotiable across all five:** every project must include an **eval** and an **observability** layer. That's not gold-plating — it's the entire difference between an AI Product Engineer and a tutorial-follower. A demo that "worked three times" is `pass@1` on a happy path; a project with a Langfuse trace showing a measured faithfulness score, or a CI badge that fails the PR on a regression, is *evidence*. Pick from the [tool stack by layer](../tools.md); you don't need all of it, one per layer you touch.

> [!TIP]
> **Depth beats breadth.** Two projects done to the eval-and-observability bar beat five demos. If you only ship two, ship **Project 1** (proves the RAG + observability core) and **Project 4** (proves you can measure). Those two answer most of a hiring loop.

---

## 1. 🔎 Chat-with-my-docs — RAG + observability

**Problem.** Answer questions over a private document set (your notes, a company wiki, a codebase's docs) with grounded, *cited* answers — and prove the answers are faithful to the source, not hallucinated.

**Stack.** LlamaIndex or LangChain (orchestration) · **pgvector** (start here — no new infra) or Qdrant (vector DB) · a hosted model API · **Langfuse** (observability) · RAGAS (retrieval eval) · Next.js (UI).

**Milestones.**
1. Ingest + chunk + embed a real corpus; hybrid retrieval (dense + BM25) with a re-ranker.
2. Grounded generation with inline citations; a "not in my docs" refusal path.
3. Instrument every retrieval + generation as a **span** in Langfuse (query, chunks, tokens, cost, latency).
4. A RAGAS eval set (faithfulness, context precision/recall) run in CI; deploy the UI.

**Proof artifact.** A deployed URL **plus** a Langfuse trace link showing a per-answer **faithfulness score** and cost/latency — the observability layer is what makes this hireable, not the chat box.

---

## 2. 🎙️ Real-time voice agent

**Problem.** A live voice assistant that handles a real task (booking, triage, Q&A) — and survives the things demos hide: a user interrupting mid-sentence, a tool call failing, an ambiguous request.

**Stack.** **LiveKit Agents** or **Pipecat** (real-time STT → LLM → TTS, barge-in) · a frontier LLM · **Arize Phoenix** (tracing + online eval) · a couple of poka-yoke tools (with structured TRANSIENT/PERMANENT/REQUIRES_HUMAN errors).

**Milestones.**
1. A working STT → LLM → TTS loop with **interruption handling** (barge-in) and turn detection.
2. One real tool call (lookup/booking) with structured errors and a retry budget.
3. A graceful **error-fallback** path when the tool or model fails (don't crash — degrade).
4. Span traces of every turn (tool, cost, latency) in Phoenix; a small trajectory eval on recorded sessions.

**Proof artifact.** A screencast demoing an interruption, a successful tool call, **and** a triggered error fallback — plus a Phoenix trace of the session. Voice makes the failure modes vivid; showing you handle them is the signal.

---

## 3. 🧠 MCP-powered second brain

**Problem.** Expose your own knowledge (notes, calendar, a personal knowledge base) to Claude/Cursor through the **Model Context Protocol**, so any MCP client can query it — and do it *safely*, treating the server as a real trust boundary.

**Stack.** MCP Python SDK (build a server exposing **tools**, **resources**, **prompts**) · your notes/calendar/KB as the backing store · Claude Code or Cursor as the client · least-privilege tool scoping + an allow-list.

**Milestones.**
1. An MCP server exposing 3–5 tools (search notes, read a doc, list calendar) over stdio.
2. Least-privilege scoping + an allow-list (no `shell_exec` on a read-only brain); version-pinned.
3. A resource + a prompt primitive, not just tools; connect it to Claude/Cursor.
4. A short eval: does the client select the right tool with the right args? Log every `tools/call`.

**Proof artifact.** The repo **plus** a screencast of Claude/Cursor querying your second brain live, and a note on the security posture (scope, pin, audit) — showing you understand MCP is an integration *and trust* boundary, not just glue.

---

## 4. 🧪 Production-style eval suite

**Problem.** Given an existing AI feature (yours or an open-source one), build the eval discipline that catches a silent regression — the thing that turns "users told us it got dumber" into "CI told us before we shipped."

**Stack.** **DeepEval** (pytest-style LLM tests) · **RAGAS** (if the target is RAG) · a binary, cross-family LLM judge · **GitHub Actions** (CI gate) · pinned model + prompt versions.

**Milestones.**
1. Error analysis on 50 real/realistic traces → cluster into failure modes → a targeted eval per dominant mode.
2. A capability eval and a regression eval; report **`pass^k`**, not `pass@k`.
3. A calibrated binary LLM judge (justification required, judge ≠ generator family, Cohen's κ measured).
4. A GitHub Actions workflow that **fails the PR on a regression** — then prove it by simulating a model swap.

**Proof artifact.** A green→red CI run showing the gate *catching* a deliberately introduced regression, and an **eval-on-PR badge** in the repo. This is the project that most directly proves "eval is the new system design."

---

## 5. 🎛️ Fine-tuned specialized model

**Problem.** Take a base model that fumbles a narrow behaviour, format, or skill (a specific output shape, a consistent voice, a domain classification) and fine-tune it — for **behaviour/format**, not to inject facts (that's RAG). The point is to prove you know *when* fine-tuning is the right lever.

**Stack.** **Unsloth** (fast LoRA/QLoRA) or **HF TRL** (SFT/DPO) · a small domain dataset · Modal or a free GPU tier · a before/after eval harness (reuse Project 4's).

**Milestones.**
1. Assemble a small, clean SFT dataset for the target behaviour; write the eval *first*.
2. Baseline the un-tuned model on your eval set (this is the "before").
3. LoRA/QLoRA fine-tune; measure the "after" on the same set.
4. Write a **model card**: what it does, the dataset, the before/after numbers, and the failure modes.

**Proof artifact.** A training notebook, a **model card**, and a **before/after eval** on your golden set. Fine-tuning without an eval is a vibe; the delta is the deliverable — and being able to say "here's why RAG would've been wrong for this" is the senior signal.

---

## The bar, in one line

For each project ask: *can a hiring manager click a link and see a **measured** result?* A deployed URL with a faithfulness score, a CI badge, a before/after eval, a traced session. If the answer is "you'd have to run my notebook," it's not done yet.

---

[⬅ Roadmap index](../README.md) · [Stage 7 — Shipping & Observability](../roadmap/7-shipping-observability-ops.md) · [Are you ready to apply? →](../readiness-check.md)
