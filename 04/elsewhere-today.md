---
title: "Elsewhere today: Fermat in Lean, a Chrome zero-day"
url: "https://bosnadev.com/2026/09/04/elsewhere-today"
author: "Mirza Pašić"
date: "2026-09-04"
topic: "Links"
tags: ["roundup"]
summary: "Claude formalizes Fermat's Last Theorem in Lean, a circuit-board benchmark that runs SPICE, a Chrome zero-day, and pricier products in Google AI Mode."
---

# Elsewhere today: Fermat in Lean, a Chrome zero-day

**[Anthropic has formalized Fermat's Last Theorem](https://www.anthropic.com/research/formalizing-fermats-last-theorem).** Claude did the work over 11 days, mostly autonomously, producing what Anthropic calls the first complete computer-verified formalization of Fermat's Last Theorem in [Lean](https://lean-lang.org/). It is not a new proof; Andrew Wiles already provided one. The work expresses that proof in a form whose logical steps can be checked mechanically. It deserves more attention than another mathematics benchmark would get.[^1]

[**Can AI now design circuit boards?**](https://www.eebench.org/blog/can-ai-design-circuit-boards-yet/) EEBench is trying to build the electronics version of executable code evaluations. Instead of judging whether a generated schematic looks plausible, it builds a circuit graph, runs [SPICE](https://en.wikipedia.org/wiki/SPICE) simulations and checks measurable criteria such as voltages, thresholds, ripple and cost. Claude Opus 5 leads its published September 1 results with 61.6%. I like this approach: once models do engineering work, the useful benchmark is whether the device works.[^2]

**[Google has patched an actively exploited Chrome zero-day](https://www.bleepingcomputer.com/news/security/google-warns-of-new-chrome-zero-day-flaw-exploited-in-attacks/).** `CVE-2026-85046` is a type-confusion bug in [V8](https://v8.dev/) that allows arbitrary code execution inside the Chrome sandbox through a crafted page. Browser security matters more for agent infrastructure now, since so many computer-use systems effectively hand a model control of a Chromium instance.[^3]

**[GPT-6 Astra is already available via OpenRouter](https://openrouter.ai/openai/gpt-6-astra).** It took about a day for yesterday's frontier launch to become one more model you can pick through a standard API. Models are getting more portable at the same time as model-specific harnesses matter more, which is an interesting tension.

[**Who is using MCP in production?**](https://news.ycombinator.com/item?id=45123339) started a useful discussion on Hacker News today.[^4] I care more about reports of real operational problems, authentication decisions, tool boundaries and deployment patterns than about another explanation of what [MCP](https://modelcontextprotocol.io/) is supposed to become.

**[The Rust React Compiler is now native in Vite](https://blog.master.dev/the-rust-react-compiler-is-now-native-in-vite).** Another small example of Rust becoming part of the tools most JavaScript developers use without caring what language they are written in.

**[Google AI Mode shows products priced 21.6% higher than traditional search does](https://productrise.app/ai-mode-pricing-study).** This connects to [my post from two days ago](https://bosnadev.com/2026/09/02/ai-search-has-an-seo-spam-problem-already) on retrieval systems and purchasing decisions. If AI search systematically shows a different set of products from conventional search, how retrieval and recommendation work is more than an SEO question.[^5]

---

### References

[^1]: [**Anthropic · “Formalizing Fermat's Last Theorem,” 4 September 2026.**](https://www.anthropic.com/research/formalizing-fermats-last-theorem) This is the original source citing Anthropic's assertion that Claude achieved the first fully computer-verified formalization of Fermat's Last Theorem following 11 days of mostly autonomous work in Lean.

[^2]: [**EEBench, “Can AI design circuit boards yet?”, 4 September 2026.**](https://www.eebench.org/blog/can-ai-design-circuit-boards-yet/) This article describes EEBench's deterministic circuit-evaluation method and shows results from its current model.

[^3]: [**BleepingComputer · "Google warns of a new Chrome zero-day vulnerability being exploited in attacks", 4 September 2026.**](https://www.bleepingcomputer.com/news/security/google-warns-of-new-chrome-zero-day-flaw-exploited-in-attacks/) Report on `CVE-2026-85046` and Google's Chrome 152 security update.

[^4]: [**Hacker News · “Ask HN: Who is using MCP in production?”**](https://news.ycombinator.com/item?id=45123339) A discussion by practitioners regarding actual MCP deployments.

[^5]: **[ProductRise · Study into Google AI Mode's pricing model](https://productrise.app/ai-mode-pricing-study).** A study that compares products found using Google AI Mode with those obtained through traditional search.
