# Stage 1 — Foundations

`🧱 Stage 1 of 7` · **Foundations** · *how LLMs actually behave, and the ground floor everything else is built on* · updated 2026-07

> **The one idea:** an LLM is **not a function**. Same input, different output; finite working memory you pay for by the token; latency that scales with what it writes. Almost every reliability, cost, and structure decision you'll make downstream — and almost every foundational interview question you'll get — is a direct consequence of those three facts.

[⬅ Roadmap index](../README.md) · **Next:** [Stage 2 — LLM App Building ➡](2-llm-app-building.md)

---

## Why this stage exists

You can ship a demo without understanding how a model generates tokens. You cannot ship a *product* — cost-controlled, low-latency, non-regressing — without it. This stage is the mechanical model of an LLM call that makes every later stage make sense: prefill vs decode, tokens as the unit of everything, the context window as scarce ordered memory, and why "it works" on a probabilistic system is a *statistical claim that needs measurement*, not a green checkmark.

An AI Product Engineer needs enough of the model layer to reason about it — **not** an ML PhD. You are not training foundation models; you are building products on top of them. But you must be fluent in how they behave, because the product's reliability is downstream of that behaviour.

---

## Key concepts

### One call = prefill + decode

A single LLM call runs in two phases. **Prefill** ingests your entire prompt in *one parallel forward pass*, building the KV cache. **Decode** then generates output tokens *one at a time*, each a forward pass attending to the growing cache.

```
prefill ── process all N input tokens in ONE parallel pass ──► build KV cache
             (cost ∝ input length; this is most of your time-to-first-token)
decode  ── token ── token ── token ── ...   (each ≈ constant per token)

TTFT   ≈ prefill time            → shrink or CACHE the PROMPT to improve it
total  ≈ TTFT + out_tokens × inter-token-latency → shrink the OUTPUT to improve it
```

> [!TIP]
> **You optimize the two latencies with *different* levers.** To cut time-to-first-token, shrink or cache the prompt (prefill). To cut total time, shrink the output, stream it, or pick a faster-decoding model. Conflating them is the most common cost/latency mistake juniors make. A 4,000-token prompt returning 50 tokens feels snappy; a 500-token prompt returning 2,000 tokens feels slow — **output length dominates total latency.**

### Everything is tokens

Models see **tokens** (sub-word BPE pieces), not words or characters. Rule of thumb: ~4 chars ≈ 1 token. Two pricing facts to memorize:

- **Output tokens cost ~3–5× more than input tokens** (decode is sequential and compute-heavy).
- A long stable prefix can be **prompt-cached** for up to ~90% off its input cost.

So the cheapest feature has a big *cached* prompt and a *short* output — exactly backwards from "shorter prompts are cheaper."

> [!WARNING]
> **Tokenization is why LLMs "can't spell" or count letters.** "How many r's in strawberry?" is genuinely hard because the model never sees the letters — it's a tokenization artifact, not a reasoning failure. Same root cause as shaky digit-by-digit arithmetic. The fix isn't a bigger model; it's giving the model a **tool** (code execution, a calculator).

### The context window is working memory — and it rots

The window (e.g. 200k tokens) is the model's working memory *for one call*. System prompt, history, retrieved docs, tool definitions, tool results, **and** the output all share that one budget. Two empirical facts every senior must know:

- **Lost in the Middle** (Liu et al., 2023): recall is U-shaped — facts at the *start* or *end* are recalled reliably; facts in the *middle* are frequently missed.
- **Context Rot** (Chroma, 2025): answer quality degrades as input grows *even in frontier models*.

The upshot: **more context can make answers *worse*, not just slower.** Treat the window as a scarce, *ordered* resource — put what matters at the edges, not the middle. (GitHub Copilot deliberately caps prompts around 6,000 characters to stay in its latency envelope.)

### Sampling: temperature 0 is greedy, not deterministic

**Temperature** rescales logits before the softmax; **top-p** samples the smallest set exceeding cumulative probability `p`. The senior trap: **temperature 0 makes decoding greedy (argmax), not deterministic.** Batched inference, GPU floating-point non-associativity, MoE routing, and silent provider updates all cause run-to-run variation. Never `assert output == expected_string` — design for variation with **schemas + evals**.

### Silent regressions are the sneaky tax

Anthropic's April 2026 Claude Code postmortem traced two months of "it got dumber" reports to *three interacting changes* shipped weeks apart (a reasoning-effort cut, a caching bug, a verbosity tweak). Individually small; together they read as broad intelligence loss. **The lesson that protects you: pin model and prompt versions, and run an eval on every change.** This is the through-line to Stage 5.

> [!TIP]
> **"Latency is perceived as quality."** Notion cut chat latency ~2s → ~350ms (4×) for 100M+ users by serving a smaller fine-tuned model on the hot path. Perplexity sustains ~200M queries/day at p50 358ms by doing cheap retrieval first and running an expensive reranker only on the top candidates. The team that shrinks what hits the model before the model runs wins on both speed and cost.

---

## The interview questions this stage answers

Be able to answer each in 60–90 seconds, mechanism first, then production implication:

- "Walk me through what happens when you call an LLM." → tokenize → prefill (parallel, builds KV cache) → decode (sequential, one token/pass) → stop. Output is a *sample*, not a return value.
- "Why is the first token slow but streaming fast?" → prefill processes the whole prompt at once (TTFT); decode is cheap per token.
- "Estimate cost and latency for this feature." → tokens × price (output ~4× input); TTFT ∝ prompt, total ∝ output. State assumptions.
- "Is temperature 0 deterministic?" → no — greedy ≠ deterministic; design for variation.
- "The doc is bigger than the window — what do you do?" → chunk + RAG / map-reduce / hierarchical, **not** "bigger model."
- "Users say it got dumber but you shipped nothing — why?" → provider silently updated the model; pin versions + eval gate.

---

## Build-it / Use-it

**Build it once** to earn the intuition, then use the library:

- Write a token counter with `tiktoken` and back-of-envelope a feature's monthly cost on a whiteboard.
- Implement `map-reduce` summarization for a document bigger than the window (summarize each chunk, fold up).
- Reproduce a tiny character-level transformer (Karpathy's "Zero to Hero") so "attention" and "the loop" stop being magic.

Then **use** managed APIs and `tiktoken` / `transformers` for everything real — you'll never hand-roll a tokenizer in production.

---

## 📚 Resources

Capped, typed, annotated. Do the first two; skim the rest as reference.

- 💻 [**rohitg00/ai-engineering-from-scratch**](https://github.com/rohitg00/ai-engineering-from-scratch) — *repo, MIT, ~37k⭐.* 20 phases / 503 lessons with a Build-it/Use-it split. The single best hands-on backbone for this whole roadmap — start here.
- 💻 [**rasbt/LLMs-from-scratch**](https://github.com/rasbt/LLMs-from-scratch) — *repo, Apache-2.0, ~60k⭐.* Build a ChatGPT-like LLM from scratch, chapter by chapter. Overkill for a product engineer, but the best way to kill the "it's magic" instinct.
- 🎬 [**Karpathy — Zero to Hero + "Let's reproduce GPT-2"**](https://youtube.com/@AndrejKarpathy) — *video.* The canonical from-scratch series. Watch "Intro to Large Language Models" first, then reproduce nanoGPT if you have a weekend.
- 🎬 [**3Blue1Brown — "But what is a GPT?"**](https://www.3blue1brown.com/topics/neural-networks) — *video.* The clearest visual intuition for transformers and attention. Watch before you touch any math.
- 🧑‍🏫 [**DeepLearning.AI — GenAI short courses**](https://www.deeplearning.ai/courses/) — *course.* Free, hands-on labs on prompting, RAG, and agents. Bite-sized and practical.
- 📘 [**Hugging Face Transformers docs**](https://huggingface.co/docs/transformers) — *docs, Apache-2.0.* The reference for tokenizers, models, and generation config. You'll live here.
- 📘 [**PyTorch tutorials**](https://pytorch.org/tutorials/) — *docs, BSD.* Only if you go deep on the model layer; most product engineers can skim.
- 🧑‍🏫 [**The Full Stack (formerly FSDL)**](https://fullstackdeeplearning.com/) — *course.* Courses on building AI-*powered products* — the product-engineering angle, not just the model.

---

## ✅ Ready for Stage 2 when you can…

- [ ] Trace an LLM call end to end (tokenize → prefill → decode → stop) and explain prefill vs decode latency.
- [ ] Estimate a feature's monthly cost and p95 latency on a whiteboard, stating your assumptions.
- [ ] Explain why temperature 0 isn't deterministic and what to do about it.
- [ ] Name Lost-in-the-Middle and Context Rot and their product implications.
- [ ] Explain how you'd catch a silent provider-side regression.

Full self-assessment: [readiness-check.md](../readiness-check.md).

---

[⬅ Roadmap index](../README.md) · **Next:** [Stage 2 — LLM App Building ➡](2-llm-app-building.md)
