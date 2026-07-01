# Readiness check — are you ready to apply?

> A per-stage self-assessment plus the questions a real hiring loop will ask. Updated **2026-07**.

[⬅ Roadmap index](README.md) · [Portfolio projects →](projects/README.md)

---

This is the honest gate. Work the [7-stage roadmap](README.md), then come here and tick the boxes *out loud* — say the answer, don't just recognise it. If you can check every box in a stage **and** answer its sample questions without notes, you're ready for that part of the loop. When the whole page is green, stop preparing and start getting **referred**.

**How to use the sample questions.** Each is real interview-shaped: pick your answer, then read the per-option explanation. The wrong options are the *plausible* wrong answers — the ones that signal you've absorbed the hype, not the engineering. Getting the *reason* right matters more than the letter.

---

## Stage 1 — Foundations

**Can you…**
- [ ] Explain why an LLM is *not a function* (same input, different output) and what that forces downstream.
- [ ] Draw the prefill/decode split and say why TTFT tracks input length while decode is cheap and sequential.
- [ ] Reason in tokens: why output tokens cost ~4× input and dominate latency.
- [ ] Explain the context window as scarce, *ordered* memory (Lost-in-the-Middle / Context Rot).
- [ ] Say why "it works" on a probabilistic system is a statistical claim that needs measurement.

<details><summary><b>Sample question</b> — the silent-regression classic</summary>

**Users report your support bot "got noticeably worse this week," but you shipped nothing. Most likely cause + safeguard?**

- ▫️ Random bad luck — LLMs vary, nothing to do — *Variation exists, but a sustained drop with no deploy is the signature of a provider-side change.*
- ✅ The provider silently updated the model; **pin the model version** and run an **eval gate on every change** — *Exactly Anthropic's own postmortem pattern — pinned versions + a golden-set eval catch silent regressions before users do.*
- ▫️ Your context window shrank — *Context limits don't silently shrink; an unannounced model update is the usual culprit.*

</details>

---

## Stage 2 — LLM App Building

**Can you…**
- [ ] Get schema-valid output *by construction* (structured outputs / constrained decoding), not try/except.
- [ ] Explain why `temperature=0` is **not** a determinism guarantee.
- [ ] Treat the system prompt as a versioned, source-controlled, eval-gated artifact.
- [ ] Name the cost/latency levers: prompt-cache the stable prefix, stream, keep output short.
- [ ] Place few-shot examples so they don't break prompt caching.

<details><summary><b>Sample question</b> — parsing that crashes in prod</summary>

**You set `temperature=0` and parse the reply with `json.loads()`. It works in dev, crashes intermittently in prod. Best fix?**

- ▫️ A bug in your parser — `json.loads` is deterministic — *The parser is fine; the input varies. The model produces different text run-to-run even at temp 0.*
- ✅ Greedy ≠ deterministic — **enforce a schema (structured outputs) and validate** — *Constrain the output so a stray token can't produce unparseable text — the durable fix.*
- ▫️ Raise the temperature so the model is more confident — *Higher temperature increases variation — the opposite of what you want.*

</details>

---

## Stage 3 — RAG

**Can you…**
- [ ] Build a retrieval pipeline end-to-end: chunk → embed → hybrid search (dense + BM25) → re-rank → generate.
- [ ] Measure retrieval with recall@k / MRR and answers with **faithfulness** (RAGAS).
- [ ] Answer "RAG or fine-tuning to answer from our wiki?" (RAG — freshness, citations, ACL).
- [ ] Diagnose a mid-context recall failure and fix it by *shrinking and ordering* context, not enlarging the window.

<details><summary><b>Sample question</b> — the 60-document stuff</summary>

**You stuff 60 docs into one prompt; the model nails facts from the first and last few but ignores a crucial one in the middle. Best fix?**

- ▫️ Move to a larger context window — *It already fit — this is mid-context recall (Lost in the Middle), not a capacity limit.*
- ✅ **Retrieve only the relevant chunks and place them at the start/end** (or use RAG) — *Shrinking and ordering beats stuffing: edges recall best, and less context means less rot.*
- ▫️ Increase temperature so it explores more of the context — *Temperature controls sampling, not attention over a long context.*

</details>

---

## Stage 4 — Agents

**Can you…**
- [ ] Define an agent mechanically: a bounded call→observe→act loop where **the harness is the product**.
- [ ] Explain why agent cost grows super-linearly (the transcript re-sends every step).
- [ ] Design a tool like a public API (poka-yoke schemas, boundaries) and return **structured** errors (TRANSIENT/PERMANENT/REQUIRES_HUMAN).
- [ ] Decide single- vs multi-agent by *who owns the write*, and justify multi- on economics.
- [ ] Name the **lethal trifecta** and the durable fix (break a leg; gate the writes) — not "a better system prompt."

<details><summary><b>Sample question</b> — the runaway bill</summary>

**Your agent runs up a large bill re-issuing slightly reworded versions of the same failing search for many steps. You already have a max-steps cap. What's missing?**

- ▫️ Nothing — the step cap will stop it — *A step cap bounds count, not spend; many cheap-looking steps with growing re-sent context are still expensive, and no progress is made.*
- ✅ **Semantic loop detection + a per-task cost ceiling + progress detection** (no new info in K steps → stop/escalate) — *Exact-match detection misses reworded calls; a cost ceiling and progress check catch the expensive, non-productive loop a step cap allows.*
- ▫️ A larger retry budget so it tries more variations — *That makes it worse — more permission to re-issue a call that isn't working.*

</details>

<details><summary><b>Sample question</b> — the injection question</summary>

**Your agent reads customer-uploaded documents and can send emails. Most durable defense against indirect prompt injection?**

- ▫️ A strong system prompt telling it to ignore malicious instructions in documents — *Prompt-level defenses cut ASR but never to zero (73.2% baseline); a determined injection gets through.*
- ✅ **Break the trifecta and gate writes**: separate the data/egress planes, least-privilege tools, human approval on sends — *Architectural separation + a write gate removes the exploit path itself, rather than hoping the model resists every injection.*
- ▫️ Lower the temperature so it's less suggestible — *Temperature doesn't govern instruction-following from retrieved content; it's irrelevant to injection.*

</details>

---

## Stage 5 — Evals & Reliability

**Can you…**
- [ ] Start an eval program from nothing via **error analysis** (read 50–100 traces, open-code, cluster into failure modes).
- [ ] Distinguish capability evals (start low, climb) from regression evals (near 100%, gate CI) and graduate one into the other.
- [ ] Grade the **trajectory** (tool choice, args, order, step efficiency), not just the final answer.
- [ ] Report **`pass^k`** and explain what a 0% pass@100 usually means.
- [ ] Build a trustworthy LLM judge: binary, criterion-specific, cross-family, human-calibrated (Cohen's κ).

<details><summary><b>Sample question</b> — the SLA metric</summary>

**You're reporting reliability for a customer-facing support agent. Which metric should the SLA use?**

- ▫️ `pass@k` — it shows the agent can succeed — *It rewards "at least one of k worked," overstating reliability for a user who gets one shot.*
- ✅ **`pass^k`** — all k attempts must succeed — *Production reliability means consistent success; 90% per attempt → 59% pass^5 is the user-facing truth.*
- ▫️ `pass@1` — simplest to compute — *A single shot hides variance; it's a smoke test, not an SLA.*

</details>

<details><summary><b>Sample question</b> — the inherited agent</summary>

**You inherit an agent with no evals and a vague "make it better" mandate. First move?**

- ✅ **Read 50–100 real traces, open-code what went wrong, cluster into failure modes, build a targeted eval for the most frequent** — *Error analysis tells you what actually breaks and how often — not metrics invented in the abstract.*
- ▫️ Add an LLM judge that rates overall quality 1–5 on every response — *A coarse quality score with no error analysis or calibration can't drive a fix and hides process failures — the classic anti-pattern.*
- ▫️ Upgrade to the newest model and re-test by hand — *You can't tell "better" from "differently broken" without an eval set.*

</details>

---

## Stage 6 — AI Product, Design & UX

**Can you…**
- [ ] Define "done" for an AI feature **distributionally** (target rate + scoped inputs + graceful tail), not a green checkbox.
- [ ] Match capability tier and autonomy to the *cost of a wrong output* (FP-heavy vs FN-heavy).
- [ ] Treat refusals and empty states as brand moments; design a redirect, not a lecture.
- [ ] Critique/design an AI feature by clarifying user/data/metric/stakes *before* diving into a screen.
- [ ] Design calibrated-trust UX: honest mental model, error taxonomy, provenance, control primitives.

<details><summary><b>Sample question</b> — the "85% accurate" trap</summary>

**"Our summarizer is 85% accurate — do we ship?" Strongest first move?**

- ✅ **Refuse the flat number**: ask the task, the cost of a wrong output, how the 15% is distributed, and the fallback path — then define a ship *gate* — *"Done" is distributional; a single accuracy number hides whether the failures are catastrophic or graceful.*
- ▫️ Yes — 85% is above most baselines — *A number with no failure-distribution or cost-of-error context can't justify a ship decision.*
- ▫️ No — wait for 95% — *An arbitrary threshold ignores stakes; a 99% feature can be un-shippable if the 1% is catastrophic and ungraceful.*

</details>

---

## Stage 7 — Shipping, Observability & Ops

**Can you…**
- [ ] Explain why observability sits *above* controlled evals in the evidence hierarchy, and instrument spans.
- [ ] Alert on **cost-per-resolved-task** and **steps-per-task**, and say why cost is the earliest regression signal.
- [ ] Run the online→offline eval loop and a shadow → canary → A/B → full rollout.
- [ ] Choose managed API vs self-host on unit economics; name the latency levers.
- [ ] Decide MCP vs in-process tools and secure a third-party MCP server (scope, pin, sandbox, audit).
- [ ] Say when to fine-tune (behaviour/format, last) vs RAG (facts), with a before/after eval + model card.

<details><summary><b>Sample question</b> — works in staging, fails in prod</summary>

**Your agent passes every staging test but misbehaves in production. Most likely cause + fix?**

- ▫️ The model is too small — upgrade it — *Model size rarely explains a staging/prod gap; the gap is in the inputs and your visibility into trajectories.*
- ✅ **Production feeds untrusted content that perturbs trajectories** — add span-level traces and a shadow-mode rollout on real traffic — *Curated staging inputs hide the trajectories real content triggers; decision-level traces + shadow mode surface them before full rollout.*
- ▫️ Add more retries — *Retries don't reveal why the agent chose the wrong path — and can mask the problem while inflating cost.*

</details>

<details><summary><b>Sample question</b> — the earliest regression signal</summary>

**Earliest signal a prompt change quietly regressed your agent?**

- ▫️ A drop in the final-answer quality score — *Quality shifts later; by then users are affected.*
- ✅ **A jump in tokens / cost per resolved task** (an extra tool call or two) — *A small tweak that adds a tool call shows up in cost before quality — and the re-sent transcript amplifies it.*
- ▫️ An increase in HTTP 500s — *That's infra; an agent regression usually leaves the service "healthy" while behaviour drifts.*

</details>

---

## The whole-loop question a hiring manager will actually ask

*"Walk me through an AI feature you shipped — how did you know it was ready?"*

A strong answer traverses the ladder without being prompted: the feature and its **failure profile** (Stage 6) → "done" defined **distributionally** with an eval built from real failures (Stage 5) → **span traces** and cost-per-resolved-task on the live distribution (Stage 7) → the **security** posture if it touches untrusted data (Stage 4) → the **unit economics** per task. If you can tell that story about one of your [portfolio projects](projects/README.md) with real numbers, you're ready.

---

## You're ready when…

- [ ] Every stage above is green — you can *say* the answers, not just recognise them.
- [ ] You've shipped **4–5 portfolio projects**, each with an eval and an observability artifact.
- [ ] You can tell the whole-loop story above about one of them, with numbers.

Then stop preparing. **Get matched, get prepped, get Landed →** [landed.jobs](https://landed.jobs)

---

[⬅ Roadmap index](README.md) · [Portfolio projects →](projects/README.md)
