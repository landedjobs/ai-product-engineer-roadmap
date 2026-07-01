# Contributing

Thanks for helping keep this roadmap the best one for the **AI Product Engineer** role. This is a curated, opinionated resource — not a link dump — so contributions are held to a quality bar. Updated **2026-07**.

[⬅ Roadmap index](README.md)

---

## What we want

- **New resources** that are genuinely better than what's already listed (more current, more concrete, better-annotated).
- **Fixes** — dead links, stale star counts/licenses, out-of-date 2026 framing, typos.
- **Sharper stage content** — a clearer explanation, a better senior-trap callout, a missing interview angle.
- **Project improvements** — a tighter brief, a better proof artifact, a clearer milestone.

Open an [issue](.github/ISSUE_TEMPLATE/) first for anything large, so we can agree on scope before you write.

---

## The quality bar for a resource (the typed-annotated-capped-link rule)

Every link in this repo must be **all four** of:

1. **Typed** — tag what it is inline: `📄 paper · 📘 docs · 🎬 video · 🧑‍🏫 course · 🛠️ tool · 💻 repo · 📰 article`.
2. **Annotated** — one sharp sentence on *why it's here and what it teaches*. Never a bare URL. The annotation should tell a reader whether to click, and what they'll get.
3. **Capped** — roughly **8 links per topic/stage**. If you're adding a ninth, it should *replace* a weaker one, not pile on. Curation is the product; a list of 40 links is a search engine, not a roadmap.
4. **License-noted** — for repos and tools, state the license and (for repos) an approximate star count, e.g. `— *repo, Apache-2.0, ~17k⭐.*`. Licenses matter because people **ship** from this roadmap.

**Format to match** (copy an existing line from any `roadmap/<n>-*.md`):

```markdown
- 💻 [**owner/repo**](https://github.com/owner/repo) — *repo, Apache-2.0, ~17k⭐.* One sentence: what it teaches and why it earns a slot.
```

A resource that isn't all four gets sent back with a request to fix it — not rejected, just not merged as-is.

---

## License discipline (read this before copying anything)

This repo ships **MIT** (see [LICENSE](LICENSE), © 2026 Landed). To keep it cleanly MIT-licensed, adapt other people's work only under compatible terms:

| Source license | What you may do |
|---|---|
| **MIT / Apache-2.0 / BSD / CC0** | ✅ Adapt freely, **with attribution** — cite `License · Source · Authors` in the annotation or a footnote. |
| **CC-BY / CC-BY-SA** | 🔗 **Link and add your own commentary** — do not copy text verbatim into this MIT repo. |
| **CC-BY-NC-SA** (e.g. [kamranahmedse/developer-roadmap](https://github.com/kamranahmedse/developer-roadmap)) | 🔗 **Reference and mimic the *structure*** — you may model a node-graph or stage layout on it, but do **not** copy its content and do **not** relicense it. |

> [!WARNING]
> The visual roadmap format is inspired by `kamranahmedse/developer-roadmap`, which is **CC-BY-NC-SA 4.0**. We mimic the *idea of a staged skill ladder* — we do **not** copy its content or relicense any of it. When in doubt, **link and write your own words**. A verbatim copy from an incompatible license is the one thing that gets a PR closed outright.

Owned Landed course content is rephrased and restructured into these files; it's ours, unencumbered. If you're contributing original writing, it's contributed under this repo's MIT license.

---

## Style

- **Voice:** sharp, senior, concrete, no fluff. Answer-first. Lead with the mechanism, then the production implication.
- **Interview-angle everywhere:** if a concept shows up in a real loop, say so.
- **Dated 2026:** reasoning models, MCP (table-stakes and a trust boundary), agentic/trajectory eval, context engineering, `pass^k`, "eval is the new system design."
- **Formatting devices:** use `> [!TIP]`, `> [!WARNING]`, `> [!NOTE]` for aha-moments and senior traps; `<details>` for long disclosures; **tables over prose** for anything comparable.
- **The README is an index**, not the whole course — deep-link into `roadmap/`, `projects/`, `tools.md`. Don't inline stage content into the README.
- **Indentation:** match the surrounding file.

---

## How to submit

1. Fork and branch.
2. Make the change; keep PRs focused (one stage, or one coherent set of resources).
3. Check your links resolve (CI runs [lychee](.github/workflows/links.yml) on every PR).
4. Open the PR using the [pull request template](.github/PULL_REQUEST_TEMPLATE.md); explain *why* your addition beats what's there.

By contributing you agree your contribution is licensed under this repo's [MIT License](LICENSE).

---

Questions? Open an issue. And if this roadmap helped you — ⭐ **star it** and pass it on.

[⬅ Roadmap index](README.md)
