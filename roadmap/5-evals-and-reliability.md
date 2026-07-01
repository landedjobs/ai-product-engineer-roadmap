# Stage 5 — Evals & Reliability

`🧪 Stage 5 of 7` · **Evals & Reliability** · *turn "looks right" into a number you can gate CI on* · updated 2026-07

> **The one idea:** for a probabilistic system, **eval is the new system design.** Without evals you can't tell "it got better" from "it quietly got worse at something it used to do." The eval set — built from real failures — is the **most valuable artifact you own**, because the prompt and the architecture are downstream of what it tells you.

[⬅ Stage 4](4-agents.md) · [Roadmap index](../README.md) · **Next:** [Stage 6 — AI Product, Design & UX ➡](6-ai-product-design-ux.md)

---

## Why this stage exists

This is the discipline that separates "I prompted an agent" from "I shipped one" — and it's the discipline Daniel Bentes names as the defining trait of the AI Product Engineer: **evaluation engineering as a first-class discipline**, ranked by an evidence hierarchy:

```
production telemetry   ▲ strongest evidence "works" means
controlled evals       │                    a MEASURED success rate
staged tests           │
demos                  ▼ weakest — a demo is survivorship bias
```

A demo that worked three times is `pass@1` on a happy path. This stage is how you climb the hierarchy.

---

## Key concepts

### Eval sets come from *error analysis*, not vibes

The hardest part isn't the harness — it's knowing *what* to measure. **Hamel Husain's method:**

```
1. collect 50–100 real/realistic traces
2. OPEN-CODE: one free-text note per trace — what went wrong, specifically
3. AXIAL-CODE: cluster notes into failure modes:
     "skipped the auth check"        42%
     "wrong tool for date math"      18%
     "looped on a 404 (treated 503)" 15%
4. build a small, targeted eval per dominant mode (start with the 42%)
5. fixes lower that mode → re-read traces → new modes surface
```

> [!WARNING]
> **The classic anti-pattern:** a single "rate quality 1–5" LLM judge with no error analysis, no failure-mode decomposition, no calibration. It's uncalibrated, too coarse to drive a fix, and blind to process failures. If you inherit an agent with no evals, **read 50 traces**, don't reach for a generic judge.

### Capability evals climb; regression evals guard

Anthropic's split: **capability evals** ask "what can it do?" and should *start low* (a mountain to climb — Claude went 40% → 80%+ on SWE-bench Verified). **Regression evals** ask "does it still handle everything it used to?" and sit near **100%**, gating CI. The lifecycle: ship a small capability eval from real failures; once it saturates, **graduate it into the regression suite.** Saturation is a signal, not success.

### Grade the trajectory, not just the answer

For agents, output-only evals bless the wrong-but-lucky, right-but-wasteful, and skipped-safety-step runs. **Notion went from 3 to 30 fixes/day** by switching to trajectory grading (*was the right tool selected, with the right arguments, in the right order?*). Braintrust's **step efficiency** makes it concrete — 7 tool calls where 3 would do is 43%, a regression output-only evals mark "success." Layer deterministic checks (tool/argument correctness) with model-based rubrics (plan quality), and read transcripts so you don't over-punish creative-but-correct paths.

### pass^k, not pass@k

The metric encodes your reliability bar:

```
pass@1   single-shot            cheap smoke test; hides variance
pass@k   ≥1 of k attempts       best-case ceiling; dev-time screening
pass^k   ALL k attempts succeed PRODUCTION reliability / SLA bar

if one attempt is 90% reliable:
  pass@5 = 1 − 0.10^5 = 99.999%  (looks amazing)
  pass^5 = 0.90^5     = 59%      (the user-facing truth)
```

Ship customer-facing agents on **pass^k**. A real user gets one try, then another — they experience the *conjunction*.

> [!TIP]
> **A 0% pass@100 almost always means a *broken task* — not an incapable agent.** Impossible grader, missing tool, wrong setup — or, on a security eval, the model *correctly refusing* a malicious request. Give judges an "Unknown" option and read the transcripts.

### LLM-as-judge: powerful, biased, calibrate it

Documented biases: **position** (first option wins), **self-preference** (self-enhancement error 16.1 vs 8.91 for two models), verbosity, authority. The playbook:

- Stop asking for a number — use **binary, criterion-specific** judges with a written pass/fail definition and a required justification.
- **Never use the same model family to generate and judge** (self-preference).
- Randomize candidate order and average; give the judge an explicit **"Unknown"** option.
- **Calibrate against human labels** — track Cohen's κ, aim for ≥ 0.6–0.8 before you trust it; re-calibrate after any rubric edit (rubric-induced preference drift is real).

### Eval-gated CI is the only defense against silent regression

Every prompt edit, tool change, model upgrade, and provider checkpoint runs the regression suite in CI; a drop below the bar blocks the change. This is the defense against the failure mode Stage 1 named — a provider silently updates the model and quality sags weeks later, with nothing in your git log to blame. **Pin the model and prompt versions, run the eval on every change**, and "users told us it got dumber" becomes "CI told us, before we shipped."

---

## The interview questions this stage answers

- "How do you start evaluating an agent with no evals?" → error analysis: read 50–100 traces, open-code, cluster into failure modes, build a targeted eval per dominant mode.
- "Capability vs regression evals?" → capability starts low and you climb it; regression sits near 100% and gates CI; graduate saturated capability evals.
- "Why grade the trajectory?" → output-only blesses wrong-but-lucky, wasteful, and skipped-safety runs (Notion: 3 → 30 fixes/day).
- "pass@k or pass^k for a customer-facing agent?" → pass^k — the user experiences the conjunction (90%/attempt → 59% pass^5).
- "How do you make an LLM judge trustworthy?" → binary criterion-specific prompts, judge ≠ generator family, randomized order, Unknown option, human-calibrated κ.
- "How do evals prevent silent regressions?" → pin versions + gate every change on the regression suite in CI.

---

## Build-it / Use-it

- **Build:** an error-analysis pass on 50 traces, a binary criterion-specific LLM judge, a `pass^k` harness, and a GitHub Actions workflow that fails the PR on a regression.
- **Use:** DeepEval (pytest-style, drops into CI), RAGAS (faithfulness / context precision-recall for RAG), Evidently (dashboards). This is the **eval/observability layer every portfolio project must include.**

---

## 📚 Resources

- 📰 [**Anthropic — Demystifying evals for AI agents**](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — *article.* Capability vs regression, trajectory grading, pass@k. The reference framing for agent evals.
- 📰 [**Hamel Husain — Your AI product needs evals**](https://hamel.dev/blog/posts/evals/) — *article.* Error analysis, binary judges, calibration. The method that turns "make it better" into a measured plan.
- 🎬 [**Hamel Husain — Why AI Evals Are the Hottest New Skill**](https://youtube.com/watch?v=BsWxPI9UM4c) — *video.* The talk that made "eval is the new system design" a hiring signal.
- 💻 [**confident-ai/deepeval**](https://github.com/confident-ai/deepeval) — *repo, Apache-2.0, ~17k⭐.* pytest-style LLM tests, G-Eval, red-teaming. The easiest eval framework to drop into CI.
- 💻 [**explodinggradients/ragas**](https://github.com/explodinggradients/ragas) — *repo, Apache-2.0, ~15k⭐.* RAG-specific metrics: faithfulness, context precision/recall. Prove your Stage 3 retrieval works.
- 💻 [**Arize-ai/phoenix**](https://github.com/Arize-ai/phoenix) — *repo, Apache-2.0, ~10k⭐.* Tracing **and** eval in one OpenTelemetry-native tool — the bridge into Stage 7.
- 📰 [**Braintrust — AI agent evaluation framework**](https://www.braintrust.dev/articles/ai-agent-evaluation-framework) — *article.* Trajectory grading and step efficiency, made concrete.
- 📰 [**Phil Schmid — pass@k vs pass^k**](https://www.philschmid.de/agents-pass-at-k-pass-power-k) — *article.* The short, sharp explainer on the reliability metric that separates demos from products.

---

## ✅ Ready for Stage 6 when you can…

- [ ] Run error analysis on real traces and build a targeted eval per dominant failure mode.
- [ ] Distinguish capability vs regression evals and graduate one into the other.
- [ ] Grade a trajectory (tool choice, args, order, step efficiency), not just the answer.
- [ ] Report pass^k and explain what a 0% pass@100 usually means.
- [ ] Build a trustworthy LLM judge (binary, cross-family, calibrated).
- [ ] Wire an eval gate into CI on pinned model+prompt versions.

Full self-assessment: [readiness-check.md](../readiness-check.md).

---

[⬅ Stage 4](4-agents.md) · [Roadmap index](../README.md) · **Next:** [Stage 6 — AI Product, Design & UX ➡](6-ai-product-design-ux.md)
