# Stage 4 — Agents & Orchestration

`🤖 Stage 4 of 7` · **Agents** · *bounded tool-using loops that are cheap, observable, and hard to hijack* · updated 2026-07

> **The one idea:** an agent is a **loop, not a personality.** The model chooses a tool, *your harness* executes it, the result re-enters context, it chooses again — until termination. **The harness is the product.** Almost every reliability bug you'll debug, and almost every agent interview question, is about the harness, not the model.

[⬅ Stage 3](3-rag.md) · [Roadmap index](../README.md) · **Next:** [Stage 5 — Evals & Reliability ➡](5-evals-and-reliability.md)

---

## Why this stage exists

A chatbot answers; an agent runs a loop. The model emits a **structured tool call** (a JSON object whose schema is the function signature — this is the Stage 2 structured-outputs idea), your code executes it, and the result feeds back as the next observation. The model is a stateless next-token predictor; the *loop* is what gives it agency, and the loop lives entirely in your code.

Three properties organize this stage:

1. **Unbounded by default** — nothing stops the model choosing tool after tool, so *you* impose a step budget and termination.
2. **Every observation re-enters context** — a tool returning a 50k-token blob crowds out the goal and degrades the next decision.
3. **Cost and latency compound per step** — the whole transcript is re-sent every step, so input tokens grow **super-linearly** with trajectory length.

---

## Key concepts

### Agent cost is quadratic-ish, not linear

```
request 1  [system + tools + user goal]              ~1,800 tok in
request 2  [ ...all of the above... + tool_result A ] ~3,400 tok in (re-sent!)
request 3  [ ...all of the above... + tool_result B ] ~6,100 tok in (re-sent!)

input tokens GROW every step → cost is roughly QUADRATIC in trajectory length.
```

> [!TIP]
> **"Why did the bill blow up when answers still looked fine?"** → the transcript is re-sent every step, so input tokens grow super-linearly. Cut steps and trim observations, and **prompt-cache the stable system+tools prefix** (re-charged at ~0.1×) — that's why you keep tool schemas *stable* across the loop.

### The agent-computer interface (ACI): design tools like a public API

Anthropic is blunt: *"optimizing tools is often more impactful than optimizing the overall prompt."* The tool surface is your real API to the model. Poka-yoke it:

- **Require enums over free strings, absolute paths over relative** — make a wrong call *unrepresentable*.
- **Return tokens, not blobs** — a 50k-token page poisons the context; return a handle or summary.
- **Pair each tool with a worked example and an explicit boundary** ("does NOT forecast").
- **Keep the set small** — past ~20–40 tools, selection accuracy drops sharply; use **tool routing** (a cheap classifier picks the 3–5 relevant tools) or namespacing.

### The cheapest reliability win: a mid-trajectory "think" step

Anthropic's **"think" tool** — a no-op that lets the model reason *after* an observation and *before* the next action — pushed `pass^1` from **0.370 → 0.570** on tau-bench Airline (a 54% relative lift).

> [!WARNING]
> The "think" tool only helps **when paired with a prompt that tells the model *what* to think about** (the relevant policy). A bare "think more" slot does much less. The 54% number is real but contingent.

### Structured tool errors so the agent can reason about failure

Return `{ ok, value, error: { kind: TRANSIENT | PERMANENT | REQUIRES_HUMAN, retry_after_ms } }`, not raw exceptions. The taxonomy carries the recovery strategy: retry TRANSIENT within budget, change approach on PERMANENT, escalate REQUIRES_HUMAN. Structured errors are a **cost lever too** — Anthropic's $124.70 agent-build analysis found much of the *waste* was retrying ambiguous failures.

### Workflow vs agent — push *down* the list

Most "agents" are a workflow in disguise. Who owns the control flow?

| Pattern | Control flow | Cost | Use when |
|---|---|---|---|
| Prompt chaining | you (fixed seq) | low | known steps |
| Routing | you (classifier) | low | distinct input types → distinct handlers |
| Parallelization | you (fan-out) | medium | independent subtasks / voting |
| Orchestrator-workers | LLM picks subtasks | med-high | subtask count unknown up front |
| Evaluator-optimizer | you (gen+critique) | medium | clear rubric, iterate-to-pass |
| **Autonomous agent** | **the model** | **HIGH** | open-ended, can't predict the path |

> [!TIP]
> When asked to "design an agent for X," **de-escalate**: "this is really a routing workflow with two handlers — I'd add an autonomous loop only if the task space is genuinely open-ended." Most "agents" are a workflow that costs 5–20× less.

### Errors compound multiplicatively

If each step is 95% reliable, a 10-step trajectory is `0.95^10 ≈ 60%`; 20 steps ≈ 36%. Long autonomous trajectories are fragile **even with a strong model.** The fixes are structural: shorter trajectories, verification gates that catch an error before it propagates, and resumable checkpoints.

### Single vs multi-agent: it's about who owns the *write*

**Cognition ("Don't Build Multi-Agents"):** parallel agents make conflicting *implicit* decisions (naming, error idioms) → Frankenstein output; a coding agent should be a single linear thread. **Anthropic's multi-agent research system:** lead + 3–5 subagents scored **+90.2%** on a research eval — at **~15× the tokens.** They're not contradicting: **multi-agent is for parallel *reads* with one serial *write*.** Shared writes → single agent. Justify multi-agent on unit economics (a $20 research report, yes; a $0.0001 chat turn, never).

### The verifier is the whole game

Self-verification *without* a real external signal is "hallucination squared" — the same model that made the error grades it. Ground the critique in a compiler, a test suite, a schema, or a calibrated human-labelled rubric. Cognition's **Devin scored 13.86%** on SWE-bench (full) — the overwhelming majority of autonomous attempts *fail*, and the system is useful only because failures are caught and recovered. **Design for the failure path, not the demo.**

### MCP — standardize tools, but treat it as a trust boundary

The **Model Context Protocol** turns M×N bespoke tool integrations into M+N: one client per model, one server per data source, exposing **tools** (model-invoked), **resources** (read-only context), and **prompts** (templates). It's an **integration/governance win, not a capability one** — use it for multi-client/vendor/team reuse; prefer in-process tools for a single latency-sensitive agent (each MCP call is an RPC/network round-trip that compounds over the loop).

> [!WARNING]
> **MCP standardizes the *attack surface* too.** Named CVEs, **tool poisoning** (a malicious tool *description* the model reads), and **rug pulls** (a server that changes behaviour after you approved it). Controls: least-privilege scope, allow-list, **pin the version** (defends against rug pulls specifically), sandbox the agent process, audit every call. At org scale, front servers with an **MCP gateway/registry** — the API-gateway pattern for agents.

---

## The interview questions this stage answers

- "What is an agent, mechanically?" → a bounded call→observe→act loop; the harness is the product.
- "Why does cost grow faster than step count?" → the transcript re-sends every step (super-linear); cache the prefix, trim observations.
- "Highest-leverage fix when it picks wrong tools?" → optimize the ACI (enums, poka-yoke, boundaries, smaller set), not the prompt.
- "Single or multi-agent?" → who owns the write? Shared writes → single agent; justify multi on ≈15× token cost.
- "When does Reflexion help?" → only with a real verifier and a bounded retry budget.
- "MCP or in-process?" → multi-client/vendor → MCP; single latency-sensitive agent → in-process.

---

## Build-it / Use-it

- **Build:** a bounded ReAct loop from scratch — step cap, loop detection (refuse to repeat `(name, args)`), structured tool errors, a mid-trajectory think step. Then a tiny MCP server exposing one tool.
- **Use:** LangGraph (durable, checkpointed graphs) or the OpenAI Agents SDK (handoffs, guardrails, tracing) for real agents; MCP for cross-client tool reuse; LiveKit/Pipecat for real-time voice.

---

## 📚 Resources

- 📰 [**Anthropic — Building Effective Agents**](https://www.anthropic.com/research/building-effective-agents) — *article.* The field's working taxonomy (workflow vs agent, the five patterns, the ACI). The single most-cited agents piece — read it twice.
- 🎬 [**Barry Zhang (Anthropic) — How We Build Effective Agents**](https://youtube.com/watch?v=D7_ipDqhtwk) — *video.* The talk version, with the "simplest thing that works" through-line.
- 💻 [**langchain-ai/langgraph**](https://github.com/langchain-ai/langgraph) — *repo, MIT, ~36k⭐.* Graph-of-nodes durable agents with state, checkpoints, and human-in-the-loop. The production default for stateful agents.
- 💻 [**openai/openai-agents-python**](https://github.com/openai/openai-agents-python) — *repo, MIT, ~28k⭐.* Lightweight agents with handoffs, guardrails, and built-in tracing. Great to read for the loop mechanics.
- 💻 [**modelcontextprotocol**](https://github.com/modelcontextprotocol) — *repo/docs, MIT.* The MCP spec + reference servers. Table-stakes by 2026 — build a server, then read the [security guide](https://modelcontextprotocol.io/specification).
- 💻 [**e2b-dev/awesome-ai-agents**](https://github.com/e2b-dev/awesome-ai-agents) — *repo, MIT, ~29k⭐.* The curated map of agent frameworks and tools — your directory when picking a stack.
- 💻 [**livekit/agents**](https://github.com/livekit/agents) — *repo, Apache-2.0.* Real-time voice/multimodal agents (interruption handling, turn detection). The backbone for the voice-agent portfolio project.
- 📄 [**ReAct: Synergizing Reasoning and Acting**](https://arxiv.org/abs/2210.03629) — *paper, Yao et al.* The canonical loop. Read it so "Thought → Action → Observation" is muscle memory.

---

## ✅ Ready for Stage 5 when you can…

- [ ] Implement a bounded loop with step cap, semantic loop detection, and a cost ceiling.
- [ ] Design a poka-yoke tool surface and explain why the ACI beats the prompt.
- [ ] Return structured tool errors and branch on the taxonomy.
- [ ] Decide single vs multi-agent on write-topology and unit economics.
- [ ] Decide MCP vs in-process and name the three MCP attack classes + their controls.
- [ ] Explain why long trajectories are fragile even with a strong model.

Full self-assessment: [readiness-check.md](../readiness-check.md).

---

[⬅ Stage 3](3-rag.md) · [Roadmap index](../README.md) · **Next:** [Stage 5 — Evals & Reliability ➡](5-evals-and-reliability.md)
