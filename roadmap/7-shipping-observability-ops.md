# Stage 7 — Shipping, Observability & Ops

`🚀 Stage 7 of 7` · **Shipping, Observability & Ops** · *close the loop — the demo becomes a monitored, cost-controlled, resumable product* · updated 2026-07

> **The one idea:** the evidence hierarchy runs **production telemetry > controlled evals > staged tests > demos**, so the last stage of the roadmap is the *first* thing a real feature needs. You don't finish an AI product by deploying it — you finish it by making it **observable**: span-level traces of every decision, cost, and failure on the real, drifting input distribution. Deploy, cost, latency, and fine-tuning all hang off that spine.

[⬅ Stage 6](6-ai-product-design-ux.md) · [Roadmap index](../README.md) · **Done? →** [readiness-check.md](../readiness-check.md)

---

## Why this stage exists

Stage 5 gave you evals — controlled measurement against fixed datasets in CI. But Daniel Bentes' evidence hierarchy puts **production telemetry above controlled evals**: the strongest possible evidence that a probabilistic feature "works" is a measured success rate *on live traffic*, because staging uses developer-curated inputs and production pulls untrusted, long-tailed content your eval suite never saw. One widely-cited figure: **~46% of agent POCs fail on exactly this observability gap** — they passed every staging test and misbehaved in production because the input distribution, not the code, is what changed.

So this stage **leads with observability**, not "build a demo and deploy it." Observability is the layer that turns "users told us it got dumber" into "our dashboard told us, before they did" — and it's the layer every portfolio project in [projects/](../projects/README.md) must include. Then, downstream of that spine: getting the model behind an endpoint (deploy/serve), controlling what a request costs and how long it takes (cost/latency), the tool-integration standard (MCP), keeping the context lean (context engineering), and — last, not first — fine-tuning for a fixed product behaviour.

---

## Key concepts

### Observability is decision-level, not service-level

Traditional APM (Application Performance Monitoring) tells you the API returned 200 with low latency and no 500s. For an AI feature that is **necessary but not sufficient** — a perfectly green APM dashboard is fully consistent with an agent that regressed badly. Your unit of debugging is the **trajectory**, not the HTTP request: you need to know *why* it chose tool A over tool B, not just that the request succeeded.

The primitive is the **span**. Instrument every operation — generation, tool call, retrieval, event — capturing the prompt, the response, token usage, latency, cost, the exact arguments, and the structured error if any. A **trace is a tree of spans**: one agent task is one trace; its generations and tool calls are child spans. Aligning to **OpenTelemetry GenAI semantic conventions** keeps the attribute names (model, tokens, tool) portable, so an agent trace lives in the same system as your service traces and you can correlate "the agent looped" with "the downstream API was slow" in one view.

```python
with tracer.span("agent.run", input=goal) as root:
    for step in range(max_steps):
        with tracer.span("generation", parent=root) as g:
            resp = model(messages, tools=TOOLS)
            g.set(tokens=resp.usage, cost=resp.cost, latency_ms=resp.ms)
        if resp.tool_call is None:
            break
        with tracer.span("tool", parent=root) as t:
            t.set(name=resp.tool_call.name, args=resp.tool_call.args)
            obs = call(resp.tool_call)
            t.set(ok=obs["ok"], error=obs.get("error"))
# a failed task score now links to the exact tool span that broke it
```

> [!TIP]
> **Cost is your earliest regression detector.** A "small" prompt tweak that nudges the agent to make one extra tool call shows up in **tokens-per-task before it shows up in answer quality** — and because the whole transcript re-sends every step, the extra call is *amplified*, not linear. Alert on **cost-per-resolved-task** and **steps-per-task**, not cost-per-call (which can stay flat while the agent makes more calls per task). Quality is a lagging indicator; by the time it moves, users already hit it.

### Online + offline evals are one loop

Offline evals (Stage 5) run your regression suite against fixed datasets in CI and **gate changes pre-ship**. **Online evals** run graders — cheap deterministic checks plus a sampled LLM judge — against *live production traces*, continuously. The flywheel: production reveals a new failure mode → you capture the trace → it becomes a permanent test in the offline regression set → the next change that reintroduces it fails CI. Observability and evals aren't two disciplines; they're a closed loop.

### Ship behind shadow → canary → A/B → full

The rollout ladder, in order, because each rung answers a question the previous couldn't:

```
shadow mode   run on real inputs, LOG outputs (don't serve)  → zero user risk, build a golden set from real failures
canary        serve 1–5% of traffic, compare vs control      → real behaviour on a slice
A/B           controlled split                                → measure the business metric
full rollout  + online-eval alarms standing watch            → cost/steps/tool-distribution drift alarms
```

The drift that bites most isn't a change *you* made. Anthropic's own postmortem traced two months of "it got dumber" complaints to **three small interacting changes** (a reasoning-effort cut, a caching bug, a verbosity tweak) — no single git commit was the culprit, and only production-distribution traces could isolate it. This is the failure mode Stage 1 named: pin your model+prompt versions, and let per-span traces on the live distribution catch what your git log can't.

### Deploy & serve: managed API first, self-host when the math flips

Most AI Product Engineers ship on a **hosted model API** and never touch a GPU — and that's correct until unit economics or latency/privacy force otherwise. When you *do* self-host (open-weights model, privacy requirement, cost at scale), the serving layer is a throughput problem: **vLLM** (PagedAttention, continuous batching) behind **BentoML** for packaging, or **Modal** for serverless GPU so you don't manage infra. Know the shape of the decision, not every knob: latency budget, tokens/sec throughput, cost-per-1k-tokens at your volume, and whether the data can leave your VPC.

### Latency: the prefill/decode split decides your UX

Time-to-first-token (TTFT) tracks *input* length (the prefill pass over your prompt, done in one parallel shot); per-token decode is cheap and roughly constant. So a 2,000-in/100-out call usually beats a 1,000-in/1,000-out one on both cost and latency, because **output tokens cost ~4× input and are generated sequentially**. The levers, cheapest first: **prompt-cache the stable system+tools prefix** (re-charged at ~0.1×), stream tokens so perceived latency drops even when total latency doesn't, trim/compress observations, cut steps, and route easy turns to a cheaper/smaller model.

### MCP: the tool-integration standard (and a trust boundary)

The Model Context Protocol turns the **M models × N tools** integration matrix into **M + N** — one client per model, one server per tool, three primitives (tools, resources, prompts). By 2026 it's table-stakes inside Claude Code and Cursor. But it's an **integration and governance win, not a capability one**: it makes the tool surface reusable and reviewable, not the agent smarter. For a single latency-sensitive embedded agent, in-process tools are lower-latency and simpler; reach for MCP when multiple clients/teams/vendors need the same tools.

> [!WARNING]
> **An MCP server is a trust boundary with real CVEs.** Tool poisoning (a crafted tool *description* smuggles instructions into the model's context), rug pulls (an approved server silently changes behaviour), and confused-deputy attacks all exist. Treat any third-party server as an untrusted neighbourhood: least-privilege scope, an allow-list, **pin the version** (the specific control against rug pulls), sandbox the process, and audit every call. At org scale, front everything with an MCP gateway. See [Stage 4](4-agents.md).

### Context engineering: the agent is what it can see

Because every observation re-enters the context and cost grows super-linearly with trajectory length, **managing what the agent sees is as important as the tools.** Four levers, in order of how often they matter: **(1) compress** observations to a summary or typed handle (`file_id`, `row_count`) instead of raw bytes; **(2) prune** stale turns once they're no longer load-bearing; **(3) externalise to disk** — the *stateless model, stateful harness* pattern: the 200k-token window is a cache, files are the database (a plan, an append-only progress log, git checkpoints), so a long task survives context resets with no lost work; **(4) keep the prefix stable** so prompt caching keeps re-charging it at ~0.1×. "Use a bigger context window" is the weak answer — more context makes the model reason *worse* (Context Rot), not just slower.

### Fine-tuning is a product decision, and it comes last

Reach for fine-tuning for **behaviour, format, or a fixed skill** — a consistent voice, a rigid output shape, a narrow classification the base model fumbles — **not** to inject volatile facts (that's RAG's job: freshness, citations, access control). It's the *last* lever, after prompting, RAG, and tool design, because it adds a training pipeline, a dataset to maintain, and a model to version. When it's genuinely the right call: **Unsloth** (2× faster, low-memory LoRA/QLoRA), **HF TRL** (SFT, DPO, GRPO), or **Axolotl** (config-driven YAML). The product artifact that proves it worked is a **before/after eval on your golden set plus a model card** — fine-tuning without an eval is a vibe, not a result.

---

## The interview questions this stage answers

- "Works in staging, fails in prod — what now?" → the input distribution changed; add decision-level span traces and a shadow-mode rollout on real traffic — not "add retries" or "upgrade the model."
- "Earliest signal a prompt change regressed the agent?" → a jump in cost-per-resolved-task / steps-per-task; the extra tool call re-sends the transcript and hits the bill *amplified*, before quality moves.
- "Why cost-per-resolved-task, not cost-per-call?" → cost-per-call can stay flat while the agent makes more calls per task; only cost-per-resolved-task tracks business value.
- "How do online and offline evals fit together?" → offline gates changes in CI; online grades live traces on the real distribution; online failures get promoted into the offline regression set — a closed loop.
- "How do you roll out a new version safely?" → shadow (log, don't serve) → canary (1–5%) → A/B → full, with alarms on cost/steps/tool-distribution.
- "MCP or in-process tools?" → in-process for a single latency-sensitive agent; MCP for multi-client/vendor/team reuse — and treat any third-party server as untrusted (scope, pin, sandbox, audit).
- "RAG or fine-tuning?" → RAG for facts (freshness, citations, ACL); fine-tune only for behaviour/format/a fixed skill, last, with a before/after eval.
- "Your agent loses the plot on a long task — why and fix?" → context rot from un-trimmed observations; compress, prune, externalise state to disk, re-inject the goal — not a bigger window.

---

## Build-it / Use-it

- **Build:** wrap one of your Stage 2–4 projects in span-level tracing, add a cost-per-resolved-task dashboard and a steps-per-task alarm, wire an online grader on sampled live traces that promotes failures into your offline set, and run it through a shadow → canary rollout.
- **Use:** **Langfuse** or **Arize Phoenix** for tracing + online eval; **vLLM + BentoML** or **Modal** if you self-host; **Unsloth / TRL** only when fine-tuning is genuinely the right lever. See the full [tools stack by layer](../tools.md).

---

## 📚 Resources

- 📰 [**Anthropic — A postmortem of three recent issues**](https://www.anthropic.com/engineering/a-postmortem-of-three-recent-issues) — *article.* Why "it got dumber" was three interacting silent changes no git commit named — the case for production-distribution traces. The canonical observability war story.
- 💻 [**langfuse/langfuse**](https://github.com/langfuse/langfuse) — *repo, MIT, ~30k⭐.* Traces, online evals, prompt management, and cost tracking; OTel-aligned. The default open-source observability + eval backbone.
- 💻 [**Arize-ai/phoenix**](https://github.com/Arize-ai/phoenix) — *repo, Apache-2.0, ~10k⭐.* Span-level trace tree **and** online eval on live traffic in one OpenTelemetry-native tool. The bridge from Stage 5 evals into production.
- 📘 [**OpenTelemetry — GenAI semantic conventions**](https://opentelemetry.io/docs/specs/semconv/gen-ai/) — *docs, Apache-2.0.* The portable attribute names (model, tokens, tool) so your agent traces aren't vendor-locked and correlate with existing service tracing.
- 📰 [**BentoML — Deploy an LLM with BentoML and vLLM**](https://www.bentoml.com/blog/deploying-a-large-language-model-with-bentoml-and-vllm) — *article.* The self-host serving path (throughput, batching, packaging) when a managed API's economics stop working.
- 📘 [**Modal — Docs**](https://modal.com/docs) — *docs, proprietary.* Serverless GPU for inference and fine-tune jobs without managing infra — the fastest path off a laptop for a portfolio project.
- 📘 [**Model Context Protocol — Docs**](https://modelcontextprotocol.io) — *docs, MIT.* The tool-integration standard and its three primitives; read the security section as a trust-boundary primer, not just an integration guide.
- 💻 [**unslothai/unsloth**](https://github.com/unslothai/unsloth) — *repo, Apache-2.0, ~68k⭐.* 2× faster, low-memory LoRA/QLoRA fine-tuning — the accessible on-ramp when behaviour/format fine-tuning is genuinely the right lever (pair with a before/after eval).

---

## ✅ The roadmap is done when you can…

- [ ] Explain why observability sits *above* controlled evals in the evidence hierarchy, and instrument an agent with span-level traces.
- [ ] Alert on cost-per-resolved-task and steps-per-task, and explain why cost is the earliest regression signal.
- [ ] Run the online→offline eval loop and a shadow → canary → A/B → full rollout.
- [ ] Choose managed API vs self-host on unit economics, and name the latency levers (cache the prefix, stream, trim, route).
- [ ] Decide MCP vs in-process tools, and secure a third-party MCP server (scope, pin, sandbox, audit).
- [ ] Keep an agent lean and resumable via context engineering and state-on-disk.
- [ ] Say when to fine-tune (behaviour/format, last) vs RAG (facts), and prove it with a before/after eval + model card.

Full self-assessment: [readiness-check.md](../readiness-check.md).

---

[⬅ Stage 6](6-ai-product-design-ux.md) · [Roadmap index](../README.md) · **You made it. →** [Are you ready to apply?](../readiness-check.md)
