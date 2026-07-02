# The AI Product Engineer stack (2026), by layer

> The canonical, license-checked map of the open-source (and a few closed) tools an **AI Product Engineer** actually reaches for — organized by the layer of the stack they live in. Updated **2026-07**.

This is the long-form companion to the summary table in the [README](README.md). Every entry is a **named tool**, a **link to official docs**, and its **license** — because "which library should I learn?" is a real question and license discipline matters when you ship.

**How to read this:** you do **not** learn all of these. Learn *one per layer you touch*, deeply — the mental model transfers, the API is a weekend. Pick by the [shape of your project](projects/README.md), not by star count.

**Legend:** 🛠️ tool · 📘 docs · 💻 repo · ⭐ approximate stars (2026) · 🟢 permissive (MIT/Apache/BSD) · 🟡 source-available/other

---

## Orchestration — assemble prompts, chains, and retrieval

The application-layer glue: prompt templates, chains, memory, retrievers, output parsers. Learn one; most concepts port.

| Tool | What it's for | Docs | License | Stars |
|---|---|---|---|---|
| 🛠️ **LangChain** | The default orchestration framework — chains, agents, retrievers, huge integration surface | [python.langchain.com](https://python.langchain.com) | 🟢 MIT | ~140k ⭐ |
| 🛠️ **LlamaIndex** | RAG-first: ingestion, indexing, query engines, and connectors for documents | [docs.llamaindex.ai](https://docs.llamaindex.ai) | 🟢 MIT | ~50k ⭐ |
| 🛠️ **DSPy** | "Programming, not prompting" — declarative modules + optimizers that compile prompts against a metric | [dspy.ai](https://dspy.ai) | 🟢 Apache-2.0 | ~36k ⭐ |
| 🛠️ **LiteLLM** | One OpenAI-shaped interface to ~100 LLM providers; a proxy so an API key never blocks you | [docs.litellm.ai](https://docs.litellm.ai) | 🟢 MIT | ~15k ⭐ |
| 🛠️ **Instructor** | Structured outputs done right — Pydantic schemas as the contract, validation + repair | [python.useinstructor.com](https://python.useinstructor.com) | 🟢 MIT | ~9k ⭐ |

> [!TIP]
> **Build-it, then use-it.** Before you `pip install langchain`, hand-write a 40-line RAG loop (embed → search → stuff → generate) and a bare tool-calling loop. You'll understand *what the framework is hiding*, and you'll debug it far faster when it leaks.

---

## Agents — tool-using loops, handoffs, real-time, and the tool protocol

The harness is the product. These give you the loop, the handoffs, the guardrails, the tracing, and the standard tool interface.

| Tool | What it's for | Docs | License | Stars |
|---|---|---|---|---|
| 🛠️ **LangGraph** | Durable, graph-of-nodes agents with state, checkpoints, and human-in-the-loop | [langchain-ai.github.io/langgraph](https://langchain-ai.github.io/langgraph/) | 🟢 MIT | ~36k ⭐ |
| 🛠️ **OpenAI Agents SDK** | Lightweight agents with handoffs, guardrails, and built-in tracing | [openai.github.io/openai-agents-python](https://openai.github.io/openai-agents-python/) | 🟢 MIT | ~28k ⭐ |
| 🛠️ **Model Context Protocol (MCP)** | The open "USB-C for agents" — one client per model, one server per tool; table-stakes by 2026 | [modelcontextprotocol.io](https://modelcontextprotocol.io) | 🟢 MIT | — |
| 🛠️ **LiveKit Agents** | Real-time voice/multimodal agents (WebRTC, interruption handling, turn detection) | [docs.livekit.io/agents](https://docs.livekit.io/agents/) | 🟢 Apache-2.0 | ~7k ⭐ |
| 🛠️ **Pipecat** | Voice-and-multimodal agent pipelines (STT → LLM → TTS, barge-in) | [docs.pipecat.ai](https://docs.pipecat.ai) | 🟢 BSD-2 | ~6k ⭐ |

> [!WARNING]
> **MCP is a trust boundary, not just convenience.** It has named CVEs, tool-poisoning, and rug-pull attacks. Treat any third-party server as an untrusted neighbourhood: least-privilege scope, allow-list, **pin the version**, sandbox the process, audit every call. See [Stage 4](roadmap/4-agents.md).

---

## Evaluation — measure quality before you ship a change

Eval is the new system design. These turn "looks right" into a number you can gate CI on.

| Tool | What it's for | Docs | License | Stars |
|---|---|---|---|---|
| 🛠️ **DeepEval** | pytest-style LLM tests, G-Eval, red-teaming; drop into CI | [deepeval.com/docs](https://deepeval.com/docs/getting-started) | 🟢 Apache-2.0 | ~17k ⭐ |
| 🛠️ **RAGAS** | RAG-specific metrics: faithfulness, context precision/recall, answer relevancy | [docs.ragas.io](https://docs.ragas.io) | 🟢 Apache-2.0 | ~15k ⭐ |
| 🛠️ **Evidently** | 100+ metrics + dashboards for LLM and ML monitoring | [docs.evidentlyai.com](https://docs.evidentlyai.com) | 🟢 Apache-2.0 | ~8k ⭐ |
| 🛠️ **Phoenix (Arize)** | Open-source tracing **and** eval, OpenTelemetry-native | [arize.com/docs/phoenix](https://arize.com/docs/phoenix) | 🟢 Apache-2.0 | ~10k ⭐ |

---

## Observability — trace every decision, cost, and failure in production

Production telemetry sits at the **top of the evidence hierarchy**. If you can only add one layer to a demo, add this one.

| Tool | What it's for | Docs | License | Stars |
|---|---|---|---|---|
| 🛠️ **Langfuse** | Traces, evals, prompt management, and cost tracking; OTel-aligned (ClickHouse, Jan 2026) | [langfuse.com/docs](https://langfuse.com/docs) | 🟢 MIT | ~30k ⭐ |
| 🛠️ **Arize Phoenix** | Span-level trace tree + online eval on live traffic | [arize.com/docs/phoenix](https://arize.com/docs/phoenix) | 🟢 Apache-2.0 | ~10k ⭐ |
| 📘 **OpenTelemetry GenAI conventions** | The portable attribute names (model, tokens, tool) so traces aren't vendor-locked | [opentelemetry.io/docs/specs/semconv/gen-ai](https://opentelemetry.io/docs/specs/semconv/gen-ai/) | 🟢 Apache-2.0 | — |

---

## Vector databases — store and search embeddings at scale

Your index — not your embedding model — usually sets your p99 latency and your RAM bill.

| Tool | What it's for | Docs | License | Stars |
|---|---|---|---|---|
| 🛠️ **pgvector** | Vectors inside Postgres — start here; no new infra, hybrid with SQL filters | [github.com/pgvector/pgvector](https://github.com/pgvector/pgvector) | 🟢 PostgreSQL | ~16k ⭐ |
| 🛠️ **Qdrant** | Rust vector DB with strong filtering, quantization, hybrid search | [qdrant.tech/documentation](https://qdrant.tech/documentation/) | 🟢 Apache-2.0 | ~26k ⭐ |
| 🛠️ **Chroma** | Batteries-included local/embedded vector store — great for prototyping | [docs.trychroma.com](https://docs.trychroma.com) | 🟢 Apache-2.0 | ~22k ⭐ |
| 🛠️ **Weaviate** | Vector DB with modules, hybrid BM25+dense, multi-tenancy | [weaviate.io/developers/weaviate](https://weaviate.io/developers/weaviate) | 🟢 BSD-3 | ~15k ⭐ |
| 🛠️ **Milvus** | Distributed vector DB built for billion-scale | [milvus.io/docs](https://milvus.io/docs) | 🟢 Apache-2.0 | ~38k ⭐ |
| 🛠️ **Pinecone** | Managed, serverless vector DB (closed source) — pay to skip ops | [docs.pinecone.io](https://docs.pinecone.io) | 🟡 proprietary | — |

> [!TIP]
> **Start with pgvector.** If your corpus fits in Postgres you already run, you get vectors, metadata filters, and transactions with zero new infra. Reach for a dedicated vector DB when your index outgrows RAM or you need billion-scale ANN.

---

## Deploy & serve — get the model behind an endpoint

| Tool | What it's for | Docs | License | Stars |
|---|---|---|---|---|
| 🛠️ **vLLM** | High-throughput LLM inference server (PagedAttention, continuous batching) | [docs.vllm.ai](https://docs.vllm.ai) | 🟢 Apache-2.0 | ~40k ⭐ |
| 🛠️ **BentoML** | Package and serve models/pipelines as production services (pairs with vLLM) | [docs.bentoml.com](https://docs.bentoml.com) | 🟢 Apache-2.0 | ~8k ⭐ |
| 🛠️ **Modal** | Serverless GPU — run inference/fine-tune jobs without managing infra | [modal.com/docs](https://modal.com/docs) | 🟡 proprietary | — |

---

## Fine-tune — adapt a model to your domain (last, not first)

Reach for fine-tuning for **behaviour, format, or a fixed skill** — not to inject volatile facts (that's RAG).

| Tool | What it's for | Docs | License | Stars |
|---|---|---|---|---|
| 🛠️ **Unsloth** | 2× faster, low-memory LoRA/QLoRA fine-tuning | [docs.unsloth.ai](https://docs.unsloth.ai) | 🟢 Apache-2.0 | ~68k ⭐ |
| 🛠️ **HF TRL** | SFT, DPO, GRPO, reward modelling — the Hugging Face RLHF/alignment toolkit | [huggingface.co/docs/trl](https://huggingface.co/docs/trl) | 🟢 Apache-2.0 | ~19k ⭐ |
| 🛠️ **Axolotl** | Config-driven fine-tuning (YAML), broad model + method coverage | [docs.axolotl.ai](https://docs.axolotl.ai) | 🟢 Apache-2.0 | ~12k ⭐ |

---

## License discipline (for contributors and forkers)

This repo ships **MIT**. When adapting other people's work:

- ✅ **MIT / Apache-2.0 / BSD / CC0** — adapt freely, with attribution (`License · Source · Authors`).
- 🔗 **CC-BY / CC-BY-SA** — link and add your own commentary; don't copy verbatim into an MIT repo.
- 🔗 **CC-BY-NC-SA** (e.g. [kamranahmedse/developer-roadmap](https://github.com/kamranahmedse/developer-roadmap)) — you may **reference and mimic the structure**, but do **not** copy its content or relicense it.

Star counts are approximate as of **2026-07** and drift; treat them as order-of-magnitude signal, not a leaderboard.

---

<div align="center">

**Missing a tool that belongs here?** [Open a resource issue →](.github/ISSUE_TEMPLATE/add-a-resource.md)

[⬅ Back to the roadmap](README.md)

</div>
