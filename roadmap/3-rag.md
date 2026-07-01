# Stage 3 — Retrieval-Augmented Generation (RAG)

`🔎 Stage 3 of 7` · **RAG** · *ground answers in trusted data — and treat retrieval as the information-retrieval problem it is* · updated 2026-07

> **The one idea:** in RAG, the model is rarely the bottleneck — **retrieval is.** If the answer-bearing chunk never makes the top-k, no model, however large, can ground an answer on it. Most production "hallucinations" are retrieval misses wearing a costume. Debug retrieval first, and measure it like the IR problem it is.

[⬅ Stage 2](2-llm-app-building.md) · [Roadmap index](../README.md) · **Next:** [Stage 4 — Agents ➡](4-agents.md)

---

## Why this stage exists

A vanilla LLM hallucinates on your internal data because the weights are a *lossy compression of the public web at training time* — no row for your Q3 board deck or yesterday's ticket. RAG is the fix that won: retrieve the relevant text *at query time* and put it in the context. That buys three things fine-tuning can't — **freshness** (re-index, don't re-train), **attribution** (cite the chunk), and **access control** (filter by who may see what).

> [!TIP]
> **"RAG vs fine-tuning?"** is a favourite interview opener precisely because the naive answer ("fine-tune on our docs") misses all three. RAG is for volatile, access-controlled *facts*; fine-tuning is for *style/format or a fixed skill*. They compose, they don't compete.

---

## Key concepts

### Embeddings + ANN — the retrieval engine

An embedding model maps text to a fixed-length vector so similar *meaning* lands at nearby points. Search embeds the query and takes the document vectors with highest **cosine** similarity (angle, not magnitude — a long doc and a short query compare on meaning, not length).

At scale you never brute-force. A **vector database** builds an **approximate-nearest-neighbour (ANN)** index that trades a few points of recall for 100–1000× speed. The dominant index is **HNSW** — a layered proximity graph with two knobs: `M` (edges per node) and `efSearch` (candidates explored). It's a **recall/latency/memory triangle you tune, not a black box.**

> [!WARNING]
> **ANN returns *approximate* neighbours — that's the "A".** At default settings HNSW might miss 1–5% of the true top-k, silently. When recall@k looks stuck, **raise `efSearch`/`M` before you blame the encoder.** The index config — not the embedding model — usually sets your p99 and your RAM bill.

### Measure retrieval before you touch the generator

You cannot debug retrieval without a small **labelled set** — ~50–200 `(query, gold-chunk-ids)` pairs that look like real traffic. Then two metrics:

- **recall@k** = fraction of queries whose gold chunk appears in the top-k. *The ceiling on everything.* Maximize this in stage 1.
- **precision@k** = fraction of the top-k that is relevant. Low precision = noise in the context. Buy this with a reranker in stage 2.

> [!TIP]
> **The first hour of any RAG project is better spent building 50 labelled queries than swapping the embedding model.** Every later decision — chunk size, hybrid on/off, encoder swap — is then an A/B test against that set. Retrieval you can't measure is retrieval you can't improve.

### Pick a *shape* before you build

The most common senior mistake is building "a RAG" without deciding which shape it is. The shape sets your latency budget, index footprint, freshness SLA, and dominant lever:

| Shape | Example | Budget | Dominant lever |
|---|---|---|---|
| **Chat-shaped** | Notion, Uber Genie | 300ms–2s, closed corpus | chunking + reranking + model choice |
| **Enterprise-search** | Glean, Harvey | sub-second p99, RBAC | the hybrid index + multi-tenancy |
| **Web-shaped** | Perplexity, feeds | seconds-fresh, billion-doc | one engine fusing lexical + dense + freshness |

Perplexity runs ~200M queries/day at p50 358ms; Notion cut chat latency ~2s → ~350ms by swapping to a smaller fine-tuned model — *not* by touching retrieval. Same word "RAG," three completely different engineering problems. **Name the shape and its freshness SLA out loud first** — it frames every later decision.

### The seven failure points (Barnett et al.)

```
1. Missing content     — the answer isn't in the corpus    → ingestion / coverage
2. Missed top-ranked   — retrieved, but below your cutoff   → reranking
3. Not in context      — right doc, wrong chunk (split)     → chunking + contextual
4. Not extracted       — in context, the model misses it    → prompt / grounding
5. Wrong format        — ignored your "return JSON"          → structured output
6. Wrong specificity   — too vague or too narrow            → query transformation
7. Incomplete          — partial answer to a multi-part Q   → decomposition
```

Failures 1–3 are **retrieval** (right text never reached the prompt); 4–7 are **generation/orchestration**. *Locate the boundary first*: if the gold chunk is in the retrieved set but the answer is wrong, no chunking or reranking helps — you're in generation territory.

### Chunking is the highest-leverage, least-glamorous decision

Your **unit of retrieval** (what you embed) is almost never your **unit of meaning** (what answers the question). Default to **~512 tokens, 10–15% overlap, recursive structure-aware splitting** — then *measure precision@5* and drop toward **256** if it sags.

> [!WARNING]
> **Bigger chunks don't "add context" — they add noise to the vector.** A 1,500-token chunk pools into *one* vector; if only 30 tokens are relevant, the signal is averaged against 1,470 irrelevant ones and the chunk ranks lower. The fix for "the model needs more context" is **small-to-big** (match small, read the parent), not bigger chunks.

**Parsing is stage zero.** Tables, scans, and multi-column PDFs corrupt text *before* any chunker runs — no chunker recovers a shredded parse. Use layout/OCR-aware extraction and keep tables as Markdown/HTML.

**Contextual Retrieval (Anthropic, 2024)** fixes the orphaned-chunk problem: before embedding each chunk, a cheap model reads the *whole document* and writes a 50–100 token context that situates it (resolving names, dates, references), indexed in *both* embeddings and BM25. The numbers stack cleanly:

```
Top-20 retrieval failure rate (lower is better)
Embeddings only ................. 5.7%
+ Contextual embeddings ......... 3.7%  (-35%)
+ Contextual BM25 (hybrid) ...... 2.9%  (-49%)
+ Reranker ...................... 1.9%  (-67%)
```

A **67% cut before you touch your model or chunk size**, at a one-time ~$1.02/M-token cost (~90% off with prompt caching).

### Hybrid retrieval — BM25 covers what dense misses

Dense retrieval is great at *meaning* but bad at *exact* tokens (error codes, SKUs, function names, surnames). **BM25** nails those but misses paraphrase. They fail on *different* queries, so fusing them wins. Fuse with **Reciprocal Rank Fusion (RRF)** — it uses *ranks*, not scores, because BM25 scores are unbounded and cosine sits in [-1, 1]. On WANDS, BM25 and dense tie alone (~0.698) but hybrid RRF jumps to 0.750 (+7.4%).

> [!TIP]
> **"Hybrid is on but exact codes still don't match" is almost always an *analyzer* bug, not a fusion bug.** A default analyzer that strips hyphens turns `E-1042` into nothing matchable. Saying that distinguishes someone who's run a lexical index from someone who's only read about RRF.

### Change chunking behind a *dual index*

Re-chunking = a full re-embed (new boundaries → new text → non-comparable vectors, "representation shearing"). Mixing generations in one index **silently shears recall** — no errors, healthy latency, every vector looks fine, but cross-generation distances are wrong. Always build the new index alongside the old, validate on a held-out set, then cut over atomically. Saying "re-chunk in place" is a red flag that you haven't run RAG at scale.

---

## The interview questions this stage answers

- "What caps a RAG system's quality?" → retrieval, not the model.
- "RAG vs fine-tuning?" → RAG for volatile, access-controlled facts (freshness, citations, ACL).
- "Define recall@k vs precision@k." → recall = did the gold chunk make top-k (the ceiling); precision = how much of top-k was relevant (noise/cost).
- "Estimate index RAM for 50M chunks." → N × dim × 4 bytes ≈ 300 GB raw at 1,536-d, × ~1.5–2 for HNSW — then mention Matryoshka/PQ to cut it.
- "A chunk says 'revenue grew 3%' with no company/quarter — fix?" → contextual retrieval; ~67% fewer top-20 failures with a reranker.
- "Change chunking on a live index — how?" → dual index, validate, atomic cutover.

---

## Build-it / Use-it

- **Build:** a 40-line RAG loop (embed → cosine search → stuff → generate), then `recall_at_k` / `precision_at_k` on a hand-built 50-query labelled set. Sweep chunk sizes against it.
- **Use:** LlamaIndex or LangChain for the pipeline, pgvector/Qdrant for the index, a cross-encoder reranker for precision, and RAGAS for faithfulness (Stage 5).

---

## 📚 Resources

- 💻 [**NirDiamant/RAG_Techniques**](https://github.com/NirDiamant/RAG_Techniques) — *repo, MIT, ~28k⭐.* One runnable notebook per technique (chunking, reranking, HyDE, GraphRAG, multimodal). The best hands-on RAG library in the ecosystem — work through it.
- 🧑‍🏫 [**DeepLearning.AI — Retrieval-Augmented Generation (Chip Huyen)**](https://www.deeplearning.ai/courses/retrieval-augmented-generation/) — *course.* Production-minded RAG from someone who's shipped it. Great mental models.
- 📰 [**Anthropic — Introducing Contextual Retrieval**](https://www.anthropic.com/news/contextual-retrieval) — *article.* The orphaned-chunk fix with the stackable 5.7% → 1.9% numbers. Required reading; the numbers are interview gold.
- 🎬 [**Jason Liu — Why Your RAG System Is Broken, and How to Fix It**](https://youtube.com/watch?v=wexpoR1R03A) — *video.* The systems-thinking view: measure retrieval, dual-index migrations, the failure taxonomy.
- 🎬 [**Greg Kamradt — The 5 Levels of Text Splitting**](https://youtube.com/watch?v=8OJC21T2SL4) — *video.* The definitive chunking walkthrough, including why semantic chunking is often not worth it.
- 💻 [**ray-project/llm-applications**](https://github.com/ray-project/llm-applications) — *repo, Apache-2.0.* Production RAG that scales each component; a great "how it looks at scale" reference.
- 📘 [**LangChain RAG tutorial**](https://python.langchain.com/docs/tutorials/rag/) — *docs, MIT.* The fastest path from zero to a working pipeline you can then dissect.
- 💻 [**ragas**](https://github.com/explodinggradients/ragas) — *repo, Apache-2.0, ~15k⭐.* RAG-specific evals: faithfulness, context precision/recall. Bridges into Stage 5 — you'll use it to prove your retrieval works.

---

## ✅ Ready for Stage 4 when you can…

- [ ] Build a labelled set and report recall@k / precision@k on your own corpus.
- [ ] Name the RAG shape and its freshness SLA before designing anything.
- [ ] Localize a wrong answer to a specific one of the seven failure points.
- [ ] Explain contextual retrieval and quote the stacked failure-rate numbers.
- [ ] Explain hybrid + RRF (ranks not scores) and the analyzer gotcha.
- [ ] Migrate chunking safely behind a dual index.

Full self-assessment: [readiness-check.md](../readiness-check.md).

---

[⬅ Stage 2](2-llm-app-building.md) · [Roadmap index](../README.md) · **Next:** [Stage 4 — Agents ➡](4-agents.md)
