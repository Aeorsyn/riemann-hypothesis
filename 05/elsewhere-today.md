---
title: "Elsewhere today: LLMs as a virus, Astra in code review"
url: "https://bosnadev.com/2026/09/05/elsewhere-today"
author: "Mirza Pašić"
date: "2026-09-05"
topic: "Links"
tags: ["roundup"]
summary: "LLM adoption modelled as an epidemic, a benchmark the models outgrew, Astra on cross-file bugs, and a lossless Vorbis and Opus recompressor."
---

# Elsewhere today: LLMs as a virus, Astra in code review

**[Large-Language Models as a Cognitive Virus](https://arxiv.org/abs/2609.03344).** Ricard Solé, David Krakauer, Michael Levin and six other researchers model LLM adoption with an epidemiological analogy, with people in one of three states: uncoupled, coupled and persistently dependent. The interesting part is the claim that social adoption can produce tipping points and lock-in, where opting out gets more expensive as more people around you use the technology. It is a theoretical model, not evidence that such a transition is happening, but it is a useful way to think about cognitive offloading.[^1]

**[Artificial Analysis has had to revise its intelligence benchmark again](https://artificialanalysis.ai/articles/artificial-analysis-intelligence-index-v4-2).** Version 4.2 adds an evaluation of agentic knowledge work and a long-context test over 4,592 PDF pages, drops GPQA Diamond because frontier models have started to saturate it, and gives more weight to private, held-out evaluations. AI benchmarks now go stale almost as fast as the models they measure, and moving to private, task-specific tests looks like the right response.[^2]

**[CodeRabbit has published early results from evaluating GPT-6 Astra in code review](https://www.coderabbit.ai/blog/gpt-6-astra-code-review-evaluation).** Astra finds about 4 per cent more actionable labelled bugs overall than GPT-5.6 Sol, and about 20 per cent more on hard cross-file bugs. It is also considerably more expensive. That fits [the model-routing argument](https://bosnadev.com/2026/09/05/the-smartest-model-should-not-do-all-the-work): the expensive model earns its cost on tasks that cross enough files and abstractions.[^3]

**[Nitter now has more working instances than before the recent takedowns](https://news.ycombinator.com/item?id=49571634).** The open-source alternative frontend for X shows how hard it is to remove decentralised software once enough people want to keep it running. I read it more as a story about the open web than about X.[^4]

[**Git hosting that will never leave Europe**](https://pushin.eu/) drew a lot of discussion today. Pushin doesn't offer a new git workflow; what it sells is European hosting and jurisdiction. Digital sovereignty used to be a procurement question. With cloud concentration, geopolitical risk and changing regulation, it is turning into a product feature.

[**Ignoring everything in `.gitignore` by default**](https://packagemain.tech/p/gitignore-everything-by-default) started a surprisingly big debate: new files would be ignored unless explicitly included. I wouldn't want that for normal application development, but it is a reminder that Git's default, that any new file might be source, comes from a quite different software environment.

[**Balrogg**](https://github.com/iczelia/balrogg) losslessly recompresses existing Vorbis and Opus audio and claims reductions of up to about 15% without decoding and re-encoding to a lossy format. Projects like this are why I still read Hacker News: a lot of interesting engineering has nothing to do with AI.

---

### References

[^1]: [**Ricard Solé, Giulio Ruffini, Francesca Castaldo, Marco Tuccio, Luis F. Seoane, Manlio de Domenico, Santiago F. Elena, David C. Krakauer and Michael Levin · “Large-Language Models as a Cognitive Virus,” 3 September 2026.**](https://arxiv.org/abs/2609.03344) The paper models transitions between uncoupled, coupled and persistently dependent users and examines tipping points, technological lock-in and reversibility using an epidemiological analogy.

[^2]: [**Artificial Analysis · “Announcing Artificial Analysis Intelligence Index v4.2,” 4 September 2026.**](https://artificialanalysis.ai/articles/artificial-analysis-intelligence-index-v4-2) Version 4.2 adds AA-Briefcase and the GDP.pdf long-context evaluation, removes GPQA Diamond after frontier models began saturating it, and increases the weighting of private held-out evaluations.

[^3]: [**CodeRabbit · “GPT-6 Astra in code review: Gains, privacy, and cost,” 4 September 2026.**](https://www.coderabbit.ai/blog/gpt-6-astra-code-review-evaluation) CodeRabbit reports approximately 4% higher actionable bug coverage than GPT-5.6 Sol overall, with the relative advantage increasing to around 20% on its harder cross-file review subset.

[^4]: [**Hacker News · “Nitter has more working instances than before the takedowns.”**](https://news.ycombinator.com/item?id=49571634) The discussion links to the Codeberg-hosted Nitter instance tracker and became one of the most-discussed stories of the day.
