---
title: "AI search has an SEO spam problem already"
url: "https://bosnadev.com/2026/09/02/ai-search-has-an-seo-spam-problem-already"
author: "Mirza Pašić"
date: "2026-09-02"
topic: "AI search"
tags: ["seo", "retrieval", "perplexity", "geo", "content-marketing"]
summary: "Three related sites published 215,128 generated “best software” pages and got 181 of 7,534 citations in a study of Perplexity’s software recommendations. What that shows, what it does not, and why retrieval is becoming the position worth competing for."
---

# AI search has an SEO spam problem already

On September 2 [Trellner Research](https://trellner.com/reports/manufactured-sources-behind-ai-recommendations/) published an analysis of the sources Perplexity cites when it recommends software. The researchers sent 760 requests to the Sonar and Sonar Pro models across 380 software categories and collected 7,534 citations from 2,055 domains. Among the sources were three websites that appear to be related and that had published **215,128 generated “best software” pages** between them.[^1]

That number makes the headline, but I think the smaller figure matters more. The three sites accounted for 181 of the 7,534 citations, about 2.4 per cent. They had not taken over Perplexity's recommendations, and the study did not show that they changed which products Perplexity recommended. It did show that publishing content at volume for machine retrieval was enough to get some of it into the evidence an AI recommendation system works from.[^1]

## Where the recommendations come from

[Perplexity's Sonar models](https://docs.perplexity.ai/api-reference/sonar-post) are built to ground answers in web content, and their API returns the search results and citation URLs behind an answer alongside the generated text. That makes them convenient to study.[^2]

Trellner defined 380 buyer-intent software categories, from CRM software to narrow vertical products, and asked both Sonar and Sonar Pro to suggest five products in each. The researchers collected the citations, checked domain popularity in [Tranco](https://tranco-list.eu/), looked at the history of suspicious sites in the [Wayback Machine](https://web.archive.org/), and published the dataset and the analysis scripts.[^1][^3][^4]

**59.8%** of the 7,534 citations pointed at domains ranked below **#100,000 in Tranco**, and **23.4%** at domains outside Tranco's top million.[^1]

> Obscure does not mean unreliable. Tranco measures popularity, and a specialised engineering blog can answer a technical question better than a large media site. Trellner makes the same distinction.

Among the frequently cited sources, [G2](https://www.g2.com/) was cited 291 times and [Reddit](https://www.reddit.com/) 261. [Guideflow](https://www.guideflow.com/) appeared 194 times, ahead of [Gartner](https://www.gartner.com/) at 158. [Wikipedia](https://www.wikipedia.org/) appeared three times.[^1]

Guideflow is the example that interests me most. It sells [interactive product-demo software](https://www.guideflow.com/product/interactive-demo) and is not a software review publication, yet the study found it cited in 96 of the 380 categories, including some it does not compete in.[^1][^5]

Nothing suggests Guideflow did anything deceptive. It did ordinary SaaS content marketing: comparison articles, category pages and other material written to attract buyers from search. An AI retrieval system then treated that material as evidence for a much wider range of software-buying questions.[^1]

## 215,128 “best software” pages

The unusual domains in the dataset were WifiTalents, WorldMetrics and Gitnux. Trellner found signs of common control: registration dates close together, the same pair of [Cloudflare](https://www.cloudflare.com/) nameservers, and very similar page structures and content taxonomies. That is circumstantial evidence, and it does not prove ownership.[^1]

The size of the operation is not in doubt. Their sitemaps listed **70,731**, **71,684** and **72,713** URLs matching this pattern:

```text
/best/<something>-software/
```

That is **215,128 pages** in total.[^1]

> There are not 215,128 meaningful software categories.

The researchers also found the sister sites ranking the same category differently, which is hard to square with editorial recommendation. Two of the sites had **“Facts & Grounding Page”** in their HTML titles.[^1]

Software buyers don't say “grounding”. People who build retrieval-augmented AI systems do: it is the step where retrieved documents go into a model's context as factual evidence. The phrase shows that at least some of these pages were written with machine retrieval in mind.

Perplexity retrieved them. The three domains appeared in 41 of the 380 categories, with 181 citations in total.[^1]

That still does not mean they controlled the results. The study shows **retrieval exposure, not recommendation manipulation**: the researchers did not remove those sources and rerun the queries to see whether Perplexity's recommendations changed.

A source has to be retrieved before it can influence an answer, and this is evidence that programmatic content at volume gets retrieved.

## Search had the same incentive

Once search engines made being found worth money, publishers optimised for whatever signals decided visibility.

A lot of SEO was useful. Clear page titles, sensible URLs, accessible HTML, descriptive headings, internal links and fast pages made the web easier to navigate for people and machines.

Then people worked out that if one useful page was valuable, 10,000 marginally useful pages might be worth more.

What followed was an adversarial loop. The engines changed their ranking, publishers reverse-engineered the change, new tactics appeared, and the engines adjusted again.

AI answer engines create a similar incentive with a different target. Traditional search, simplified:

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 920 160" role="img" aria-label="crawl → index → rank → click">
  <defs>
    <marker id="flow-arrow" viewBox="0 0 8 8" refX="7" refY="4" markerWidth="8" markerHeight="8" orient="auto">
      <path
        d="M1 1 L7 4 L1 7"
        fill="none"
        stroke="var(--primary)"
        stroke-width="1.5"
        stroke-linecap="square"
        stroke-linejoin="miter"
      />
    </marker>
  </defs>

  <g
    fill="var(--card)"
    stroke="var(--border)"
    stroke-width="1"
    vector-effect="non-scaling-stroke"
  >
    <rect x="20" y="48" width="170" height="64" />
    <rect x="255" y="48" width="170" height="64" />
    <rect x="490" y="48" width="170" height="64" />
    <rect x="725" y="48" width="175" height="64" />
  </g>

  <g fill="var(--primary)">
    <rect x="20" y="48" width="4" height="64" />
    <rect x="255" y="48" width="4" height="64" />
    <rect x="490" y="48" width="4" height="64" />
    <rect x="725" y="48" width="4" height="64" />
  </g>

  <g
    fill="none"
    stroke="var(--primary)"
    stroke-width="1.5"
    vector-effect="non-scaling-stroke"
    marker-end="url(#flow-arrow)"
  >
    <path d="M190 80 H239" />
    <path d="M425 80 H474" />
    <path d="M660 80 H709" />
  </g>

  <g
    fill="var(--card-foreground)"
    font-family="var(--font-sans)"
    font-size="18"
    font-weight="500"
    text-anchor="middle"
    dominant-baseline="middle"
  >
    <text x="105" y="80">crawl</text>
    <text x="340" y="80">index</text>
    <text x="575" y="80">rank</text>
    <text x="812.5" y="80">click</text>
  </g>
</svg>

For AI answer engines it is closer to:

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 920 160" role="img" aria-label="crawl → retrieve → cite → recommend">
  <defs>
    <marker id="process-arrow" viewBox="0 0 8 8" refX="7" refY="4" markerWidth="8" markerHeight="8" orient="auto">
      <path
        d="M1 1 L7 4 L1 7"
        fill="none"
        stroke="var(--primary)"
        stroke-width="1.5"
        stroke-linecap="square"
        stroke-linejoin="miter"
      />
    </marker>
  </defs>

  <g
    fill="var(--card)"
    stroke="var(--border)"
    stroke-width="1"
    vector-effect="non-scaling-stroke"
  >
    <rect x="20" y="48" width="170" height="64" />
    <rect x="255" y="48" width="170" height="64" />
    <rect x="490" y="48" width="170" height="64" />
    <rect x="725" y="48" width="175" height="64" />
  </g>

  <g fill="var(--primary)">
    <rect x="20" y="48" width="4" height="64" />
    <rect x="255" y="48" width="4" height="64" />
    <rect x="490" y="48" width="4" height="64" />
    <rect x="725" y="48" width="4" height="64" />
  </g>

  <g
    fill="none"
    stroke="var(--primary)"
    stroke-width="1.5"
    vector-effect="non-scaling-stroke"
    marker-end="url(#process-arrow)"
  >
    <path d="M190 80 H239" />
    <path d="M425 80 H474" />
    <path d="M660 80 H709" />
  </g>

  <g
    fill="var(--card-foreground)"
    font-family="var(--font-sans)"
    font-size="18"
    font-weight="500"
    text-anchor="middle"
    dominant-baseline="middle"
  >
    <text x="105" y="80">crawl</text>
    <text x="340" y="80">retrieve</text>
    <text x="575" y="80">cite</text>
    <text x="812.5" y="80">recommend</text>
  </g>
</svg>

As agents start acting on behalf of users, it may become:

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 920 160" role="img" aria-labelledby="crawl-retrieve-decide-act-title crawl-retrieve-decide-act-desc">
  <title id="crawl-retrieve-decide-act-title">crawl → retrieve → decide → act</title>
  <desc id="crawl-retrieve-decide-act-desc">crawl leads to retrieve, retrieve leads to decide, and decide leads to act.</desc>

  <defs>
    <marker id="crawl-retrieve-decide-act-arrow" viewBox="0 0 8 8" refX="7" refY="4" markerWidth="8" markerHeight="8" orient="auto">
      <path d="M1 1 L7 4 L1 7" fill="none" stroke="var(--primary)" stroke-width="1.5" stroke-linecap="square" stroke-linejoin="miter" />
    </marker>
  </defs>

  <g fill="var(--card)" stroke="var(--border)" stroke-width="1" vector-effect="non-scaling-stroke">
    <rect x="20" y="48" width="170" height="64" />
    <rect x="255" y="48" width="170" height="64" />
    <rect x="490" y="48" width="170" height="64" />
    <rect x="725" y="48" width="175" height="64" />
  </g>

  <g fill="var(--primary)">
    <rect x="20" y="48" width="4" height="64" />
    <rect x="255" y="48" width="4" height="64" />
    <rect x="490" y="48" width="4" height="64" />
    <rect x="725" y="48" width="4" height="64" />
  </g>

  <g fill="none" stroke="var(--primary)" stroke-width="1.5" vector-effect="non-scaling-stroke" marker-end="url(#crawl-retrieve-decide-act-arrow)">
    <path d="M190 80 H239" />
    <path d="M425 80 H474" />
    <path d="M660 80 H709" />
  </g>

  <g fill="var(--card-foreground)" font-family="var(--font-sans)" font-size="18" font-weight="500" text-anchor="middle" dominant-baseline="middle">
    <text x="105" y="80">crawl</text>
    <text x="340" y="80">retrieve</text>
    <text x="575" y="80">decide</text>
    <text x="812.5" y="80">act</text>
  </g>
</svg>

That last step is why I think this is more than another set of SEO techniques.

## Ranking first may no longer be the valuable position

Search Google for "best transactional email provider" and you get a page of results. You open a few, search Reddit, look at [Postmark](https://postmarkapp.com/), [Resend](https://resend.com/) and [Amazon SES](https://aws.amazon.com/ses/), compare prices and decide. Google's ranking matters, but there are several points where you can disagree with it.

Give the same task to an AI assistant:

> I need to find an appropriate transactional email provider for my SaaS company since we send about 100,000 emails each month; I should compare their pricing, deliverability, and handling of EU data before making a recommendation.

The assistant turns an hour of browsing into a short answer built from a few retrieved sources. It may list five citations under the answer; most users won't check them.

Change the request again:

> Choose the best option and set it up for this project.

Now retrieval is part of the purchase decision.

The valuable position may no longer be number one on Google. It may be **being one of the documents the agent retrieves when it decides what to do**.

That is why I am wary of calling GEO (generative engine optimization) "SEO for ChatGPT." The two overlap, but shaping the information a machine uses to make a recommendation is a different problem from optimising a page a person might click.

## Ordinary content marketing is more interesting than the spam

The report's headline number is **215,128** pages, but Guideflow is the more useful example, because nothing unusual is needed to explain it.

A SaaS company publishes useful content in its category to attract search traffic. An AI retrieval system uses those pages as evidence for dozens of software-buying questions. Competitors see that company suggested by [ChatGPT](https://chatgpt.com/), [Perplexity](https://www.perplexity.ai/) or [Gemini](https://gemini.google.com/) and start looking at where those recommendations come from.

Then they publish more comparison pages, competitor and alternatives pages, integration pages, benchmark reports, datasets and industry statistics. They state product facts more explicitly and try machine-readable formats. Agencies sell services that promise more brand mentions in AI answers. Tools appear that track which sources the models cite and which brands they recommend.

> Some of this will improve the web. Some of it will be awful.

Nobody has to organise **“GEO spam”**. The incentive is enough: once companies see that being in an AI system's retrieval corpus affects discovery or purchases, they will publish for it.

That is roughly how SEO became an industry.

## Limits of the study

The study used one retrieval system, on one day, with one set of prompts. It did not cover [ChatGPT](https://chatgpt.com/), [Gemini](https://gemini.google.com/), [Claude](https://claude.ai/), [Google AI Mode](https://www.google.com/) or other answer engines.[^1]

The 380 categories were written for the study, not taken from real user queries. A set that deliberately includes obscure categories will surface more obscure sources than mainstream consumer queries would.

Sonar and Sonar Pro are not fully independent either. Trellner found considerable overlap between their citation sets, which suggests the two models share much of the same retrieval infrastructure.[^1]

Each prompt ran once, so the study does not measure how stable the citations are across repeated queries. And because Tranco measures popularity rather than reliability, low-ranked domains are not proof of poor sources.[^1][^3]

Most importantly, there was no counterfactual. The researchers did not block the three programmatic sites and check whether Perplexity's recommendations changed, so we cannot say those sources materially influenced which products were chosen.[^1]

The experiment I would like to see next: run the same categories repeatedly on Perplexity, ChatGPT, Gemini, Claude and Google's AI products. Record the recommendations and citations, remove whole classes of sources, and check whether the recommendations change. Repeat over several weeks.

> That would show whether being cited translates into influence over the recommendation.

## The useful version of GEO is probably boring

One reading of this study is that companies should publish hundreds of thousands of generated pages, since AI systems retrieve them.

That might work for a while. I would not build a business on it.

A recommendation system that keeps answering from poor or fabricated sources loses its users' trust. [Perplexity](https://www.perplexity.ai/), [OpenAI](https://openai.com/), [Google](https://www.google.com/) and the others running retrieval systems have every reason to build better signals for source quality, as search engines did.

The boring alternative is to make the information you actually have **easy for machines to retrieve and hard for them to misunderstand**.

Publish your real prices. Keep documentation current. Use stable URLs. Describe product capabilities plainly instead of burying them in marketing copy. Publish original datasets and say how they were collected. Make benchmark methodology reproducible. Link each claim to primary evidence. Use structured data where it matches the content underneath.

State your product's limits too. If a feature is only on the enterprise plan, say so. If you ran a performance test, give enough detail for someone else to repeat it.

All of this helps human readers as well, which is usually a sign you are improving the information and not gaming it.

## A new kind of reader

I am still sceptical of the fast-growing vocabulary: AEO, GEO, LLMO and whatever acronym comes next. Much of it is existing marketing services repackaged for a new distribution channel.

> There is a real technical change behind the jargon, though.

For most of the web's history, publishers wrote pages for people and optimised them so machines could help people find them. Now machines read those pages themselves, work through the information and give a conclusion without anyone visiting the source.

Pages still have human readers. They now have another one: **the retrieval system that decides which documents become context for a model.**

The more agents can act, the more that retrieval decision matters. The documents an agent trusts can decide the software it recommends, the vendor it contacts, the API it integrates or the product it buys on the user's behalf.

That makes retrieval an opportunity and a point of attack.

Trellner's research does not show that AI recommendations are already thoroughly gamed. It shows the incentives, and some of the mechanisms.

> The web took about twenty years to learn how to manipulate search rankings. I would be surprised if learning to manipulate AI retrieval takes another twenty.

---

### Research data

Trellner states that the **full dataset, citations, recommendations, Tranco and Wayback lookups, vendor-liveness checks, and analysis scripts** used for the report are published alongside the study under **CC BY 4.0**.[^1]

- [**Primary report and research package**](https://trellner.com/reports/manufactured-sources-behind-ai-recommendations/)
- [**Perplexity Sonar response documentation**](https://docs.perplexity.ai/api-reference/sonar-post)

### References

[^1]: [**Trellner Research · “Three sites made 215,128 ‘best software’ pages for AI. Perplexity cites them.”**](https://trellner.com/reports/manufactured-sources-behind-ai-recommendations/) Published September 2, 2026. Primary source for the experiment, methodology, dataset, citation counts, Tranco analysis, Guideflow findings, the three programmatic domains, shared-infrastructure evidence and study limitations.

[^2]: [**Perplexity · Sonar API documentation.**](https://docs.perplexity.ai/api-reference/sonar-post) The response schema exposes both `citations` and `search_results`, including source URLs, titles and dates.

[^3]: [**Tranco · A Research-Oriented Top Sites Ranking Hardened Against Manipulation.**](https://tranco-list.eu/) Tranco maintains a reproducible ranking of one million popular domains and supports historical domain-rank lookups.

[^4]: [**Internet Archive · Wayback Machine.**](https://web.archive.org/) Used in the Trellner analysis to inspect historical captures and estimate when cited domains first appeared on the web.

[^5]: [**Guideflow · Interactive Demo product.**](https://www.guideflow.com/product/interactive-demo) Guideflow describes itself as a platform for creating interactive product demos, sandboxes and other self-guided product experiences.
