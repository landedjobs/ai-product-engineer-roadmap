# Stage 6 — AI Product, Design & UX

`🎨 Stage 6 of 7` · **Product / Design / UX** · *product sense under probability, and the interaction patterns that make AI trustworthy* · updated 2026-07

> **The one idea:** the thing that makes an AI Product Engineer different from an AI Engineer is **product and design sense on probabilistic systems.** An AI feature is a **distribution, not a binary** — so "done" is a *target rate on a scoped input class plus a graceful tail*, and **the recovery UX is the product.** The error screen isn't an edge case; on a system that's wrong on a schedule, it's the main experience.

[⬅ Stage 5](5-evals-and-reliability.md) · [Roadmap index](../README.md) · **Next:** [Stage 7 — Shipping, Observability & Ops ➡](7-shipping-observability-ops.md)

---

## Why this stage exists

You can build a technically-correct RAG pipeline and still ship a product users don't trust. This stage is the half of the role that the pure-engineering roadmaps skip: **where to point the model, how to scope it, how to spec it, and how to design the interaction** so that a probabilistic system feels reliable. It draws on three disciplines — AI PM (technical fluency, product sense, eval/launch), and AI design (human-AI interaction, prompt/context as design material, AI design patterns).

---

## Part A — Product sense under probability

### Scope is the master lever

The AI MVP is *"the smallest workload where you can measure model quality."* Scope narrows on four levers (NN/g): **inputs, outputs, tasks, subjects.** Duolingo scopes hard (one language, lesson templates, expert-reviewed); the anti-pattern is **"a chat box pasted onto your app"** — broad on every lever.

> [!WARNING]
> **"If any answer is 'anything,' you haven't scoped — you've shipped a liability."** A scoped feature you can measure beats a general one you can only hope about.

### Match capability tier and autonomy to the cost of error

The LLM is *one stochastic step inside an otherwise deterministic pipeline* — deterministic retrieval/tools fetch facts, the model reasons, deterministic validators check output. Match the **capability tier** to the **cost of error**, and gate the *irreversible* step (charge a card, send an email, delete) behind a human confirmation.

**Compound failure** is the number to carry: if each step is 95% reliable, ~10 steps ≈ **60%** end-to-end. Shorten the chain, raise per-step reliability, and **bound the blast radius** (drafts, not autonomous actions).

### RAG vs fine-tuning vs API — the question you can't fumble

The default order is a *ratchet you climb only when forced*: **PROMPT → RAG → FINE-TUNE → CUSTOM**, each gated by evals. IBM's decisive test: **"if the answer changes because the world changes, fine-tuning is the wrong lever and RAG is right."** Named proof points: **Notion** bought the model but *built the data substrate* (20B → 200B+ block rows); **Harvey** custom-trained on ~10B tokens of case law for an 83% factual-response lift and 97% attorney preference — because RAG's retrieve-and-cite failure mode *was* the bottleneck.

### The metric stack — one number lies

Never collapse "model is good" and "product is good" into one score; they can move in opposite directions. Use **one dominant metric per layer** plus a **divergence alarm**:

| Layer | Example metrics |
|---|---|
| Business / North Star | task completion, deflection, LTV |
| Product engagement | DAU/WAU, retention, time-to-task |
| Model quality | hallucination rate, faithfulness, language adherence |
| System / reliability | **p95** latency, $/task, escalation rate |

> [!TIP]
> **The divergence alarm** (KORE1's probe): "the north-star metric goes up — how would you know the model is quietly getting worse anyway?" → a held-out per-slice quality gate that runs *independently* of the product metric and blocks promotion on a faithfulness drop, regardless of DAU. Watch the **escalation-to-human rate** and **human-correction rate** as leading signals.

Metrics that mislead: raw accuracy (stratify the errors — 85% overall can be 100% on easy 80% + 25% on hard 20%), **average** latency (quote **p95**), and the **thumbs-up rate** (the EMBER study, Lee et al., NAACL 2025 — evaluators over-reward confidently-worded answers, so confidently-wrong hallucinations collect thumbs-up).

### AI incidents have legal teeth

**Moffatt v. Air Canada (2024):** a chatbot misquoted bereavement-fare policy; the tribunal ordered damages and rejected "the bot is responsible for itself." Consequence: ship customer-facing GenAI only with retrieval grounding, a confidence threshold below which it **escalates to a human**, and citation-existence checks as a launch-gate eval.

---

## Part B — Designing human-AI interaction

### The four canonical frameworks — ship their *intersection*

- **Google PAIR** (People + AI Guidebook) — mental models + calibrated trust; the *why*.
- **Microsoft HAX Toolkit** — 18 evidence-based guidelines in 4 phases (Initially / During / When Wrong / Over Time), validated in **Amershi et al., CHI 2019**; the *checklist*.
- **NN/g** — usage patterns and anti-patterns; the *empirical spine* (what NOT to ship).
- **IBM Design for AI / Carbon for AI** — six ethics pillars + the only library shipping actual components; the *primitives*.

> [!TIP]
> The unit you ship is the **intersection**, not the union: PAIR for the why, HAX for the checklist, NN/g for the anti-patterns, IBM/Carbon for the components. Single-source design review accrues invisible UX debt.

### The real interface is the model in the user's head

Most AI UX failures are **mental-model failures** — the user's private belief about what the AI can and can't do. Seed an honest model in onboarding/empty states with **capability chips** ("Drafts replies · Won't send on your behalf · Can be wrong on dates" beats "AI Assistant") and, counter-intuitively, a **deliberate failure demo**. Fluency inflates the model (Jakesch, PNAS 2023 — people can't detect AI-generated text), and a single confident miss can be a $100B event (Google Bard's James Webb error).

> [!WARNING]
> **A static, always-on "AI can be wrong" banner is the canonical non-answer** — persistent banners fade into chrome (NN/g: only 7 of 153 error cases carried any uncertainty signal). Expectation-setting is an *active, moment-of-relevance* job, activated at the low-confidence moment and scaled to stakes.

### Design the error screen first

AI is wrong *on a schedule*, and the wrong 1% clusters on adversarial/out-of-distribution inputs wearing the same confident formatting. Reach for the **four-family error taxonomy** — false positive, false negative, hallucination, low-confidence — each with a *different* UX move. For consequential surfaces (money, rights, health, time), the five-part stack: **(1) show confidence, (2) cite sources, (3) allow regenerate, (4) allow human review, (5) allow exit.** Galactica had none (pulled in 48h); Air Canada had none ($812.02).

### Calibrate trust, don't maximize it

The brief is **calibrated trust** (Lee & Moray, 1992), not maximal. Two failure modes: **over-reliance / automation bias** (Buçinça et al., 2021) and **under-reliance / algorithm aversion** (Dietvorst et al., 2015 — users abandon an algorithm after seeing it err *once*). The measured anti-over-reliance move is a **cognitive forcing function** — make the user commit *before* the AI reveals its answer (Copilot's "take the task, then suggest"). Transparency is plural — **explanation, citation, show-the-work** — and stakes-dependent; stream it at the foraging moment.

### Provenance is three modes, not one badge

| Mode | User decision | Design |
|---|---|---|
| **Authored** (AI made it, no edit) | present or reject whole artifact | persistent "Generated by AI" + C2PA credentials |
| **Suggested** (draft/ranking) | edit / accept / reject | variant-aware container + revert-to-AI button |
| **Executed** (AI acted) | confirm or undo | confirmation scaled to blast radius + audit log |

> [!WARNING]
> **A single badge collapses three decisions and three rollback costs.** Labelling an *executed* action "Suggested" is a **trust bug**, not a copy nitpick. Reserve one icon (the sparkle) for generative *output* only.

### Control primitives + the commitment axis

Automation creates a heavier "commit moment" than doing the work yourself — so you owe at least one of **undo / edit / override** per AI action, weighted to reversibility. Infer the ephemeral (Copilot ghost text: Tab/Esc), gate the irreversible (confirm + undo + a global off-switch). And **feedback is control**: binary thumbs throws away the *why* — pair accept/correct/explain, and never penalize negative feedback (ChatGPT once made a thumbs-down cost a rate-limited regeneration).

---

## Part C — Prompt & context as a design material

The **system prompt is simultaneously UI copy, brand voice, and executable behaviour** — the highest-leverage design surface, versioned like a design token (Claude's one anti-sycophancy clause changed its perceived personality product-wide with no screen change). Design it slot by slot (identity, instructions, voice, examples, output contract), each with an owner and an eval. **Refusals and empty states are brand moments, not errors** — a refusal is a *redirect* (name the boundary once, skip the lecture, substitute the real goal). And **voice is per-surface**: TTS forbids ellipses, parsers demand plain text, chat avoids lists that an artifact encourages.

> [!TIP]
> **Write the rubric before you write the prompt.** A probabilistic feature has no single correct output, so deterministic metrics mislead. Split **Core** criteria (zero-tolerance gates: no toxicity, no fabricated facts stated as certain) from **feature-specific** ones (voice, hedging, cites-a-source), calibrate two reviewers on 5–10 outputs, and tie each metric to a **user-observable property** (override rate, accepted-without-edit).

---

## The interview questions this stage answers

- "Our summarizer is 85% accurate — do we ship?" → refuse the flat number: ask the task, the cost of a wrong output, how the 15% is distributed, and the fallback path; then define a ship *gate*.
- "How is 'done' defined for an AI feature?" → distributionally: "80% of category X meets bar Y; the other 20% degrades gracefully" — not a green checkbox.
- "RAG or fine-tuning to answer from our wiki?" → RAG (freshness, citations, ACL); fine-tune only for style/format or a fixed skill.
- "Critique this AI feature / design one for X." → *don't dive into a screen* — clarify user/data/metric/stakes first, then run the four-pillar review + timeline walk (name the "When Wrong" gaps).
- "The north-star is up — how do you know the model isn't secretly worse?" → an independent per-slice quality gate + escalation/correction-rate monitors.

> [!WARNING]
> **The top weak tell in an AI-design interview is diving straight into a solution.** Opening with a screen signals you treat AI as decoration on a UI rather than a probabilistic system whose UX is determined by its *failure profile*. Open with "FP-heavy or FN-heavy?" and name the tradeoff you *didn't* take.

---

## Build-it / Use-it

- **Build:** a capability-to-feature map (one row per feature: user job & error-cost · capability tier · architecture · scope · cost/latency · failure design · eval) and a redesign of one real AI feature scored on the 10-dimension audit rubric.
- **Use:** shadcn/ui AI chatbot components as a starting kit; a Wizard-of-Oz mock to validate the interaction before you wire the model.

---

## 📚 Resources

- 📘 [**Google PAIR — People + AI Guidebook**](https://pair.withgoogle.com/guidebook/) — *guide.* Mental models, calibrated trust, feedback loops. The foundational "why" of human-AI design — read Mental Models and Explainability + Trust first.
- 📘 [**Microsoft HAX Toolkit — 18 Guidelines for Human-AI Interaction**](https://www.microsoft.com/en-us/haxtoolkit/ai-guidelines/) — *guide.* The enforcement-friendly checklist, validated in [Amershi et al., CHI 2019](https://www.microsoft.com/en-us/research/publication/guidelines-for-human-ai-interaction/) (~4k citations). Grep every AI feature against it.
- 📰 [**NN/g — AI Hallucinations (design implications)**](https://www.nngroup.com/articles/ai-hallucinations/) — *article.* The empirical anti-patterns and the five hallucination-mitigation UX patterns. The "what not to ship" spine.
- 📰 [**Vitaly Friedman — Design Patterns for AI Interfaces (2026)**](https://maven.com/web-adventures/design-patterns-ai-interfaces) — *course/article.* The most current, concrete catalog of shipped AI UX patterns. Great for the pattern vocabulary.
- 📰 [**Thinking past the cliché of LLM design patterns**](https://uxdesign.cc/thinking-past-the-cliche-of-llms-ai-design-patterns-c9b849fce9e8) — *article.* Pushes past the chat-box default toward intent- and outcome-shaped interaction.
- 🛠️ [**shadcn/ui — AI chatbot components**](https://ui.shadcn.com/) — *tool, MIT.* Production-ready, brand-alignable primitives (v0 defaults to these because they help models generate real interfaces). Your prototyping kit.
- 📰 [**Anthropic — Demystifying evals for AI agents**](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — *article.* The bridge from product sense to a measurable ship gate — "each task has its own success rate."
- 📰 [**Lenny's Newsletter — AI-feature essays**](https://www.lennysnewsletter.com/) — *newsletter.* The best running case studies on shipping AI features (Notion, Perplexity, Intercom) with real numbers.

---

## ✅ Ready for Stage 7 when you can…

- [ ] Define "done" for an AI feature distributionally (target rate + scoped inputs + graceful tail).
- [ ] Scope a feature on inputs/outputs/tasks/subjects and match capability tier + autonomy to cost-of-error.
- [ ] Answer "RAG vs fine-tuning vs API" crisply, with the default ordering.
- [ ] Build a layered metric stack with a divergence alarm; quote p95 not the mean.
- [ ] Run the four-pillar design review + timeline walk and name the "When Wrong" gaps.
- [ ] Design calibrated-trust UX: honest mental model, error taxonomy, three-mode provenance, control primitives.

Full self-assessment: [readiness-check.md](../readiness-check.md).

---

[⬅ Stage 5](5-evals-and-reliability.md) · [Roadmap index](../README.md) · **Next:** [Stage 7 — Shipping, Observability & Ops ➡](7-shipping-observability-ops.md)
