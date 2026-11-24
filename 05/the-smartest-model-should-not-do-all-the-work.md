---
title: "Spotify’s Portal cut Claude Code token use by about 90% by routing work to a cheaper model"
url: "https://bosnadev.com/2026/09/05/the-smartest-model-should-not-do-all-the-work"
author: "Mirza Pašić"
date: "2026-09-05"
topic: "Agents"
tags: ["claude-code", "model-routing", "cost", "harness", "spotify"]
summary: "Spotify cut Claude Code token use on bulk reads by about 90% with a cheaper worker model. Most agent work needs no frontier model; its attention is scarce."
---

# Spotify’s Portal cut Claude Code token use by about 90% by routing work to a cheaper model

[Spotify Engineering](https://engineering.atspotify.com/2026/9/portal-by-spotify-cut-my-claude-code-token-usage-by-90) describes a simple way to cut [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) token use by about 90% for one type of task. The percentage matters less than what it shows: most of the work a coding agent does needs very little intelligence.

A coding agent spends much of its time reading files, gathering context, copying established patterns, generating repetitive tests and producing boilerplate. The frontier model contributes little reasoning to these tasks, and they can use an enormous number of tokens.

Spotify splits those tasks off. [Portal](https://github.com/spotify/portal-ai-plugins) has two worker modes: a `bulk-reader` that reads many files and returns a short summary, and a `code-writer` that produces predictable code such as tests, configuration scaffolding and type stubs. The examples use Gemini 2.5 Flash as the worker model, and Claude keeps the tasks that need more reasoning.[^1]

The architecture is roughly:

```text
                    ┌─ bulk reads ─────→ cheaper model
Claude Code ────────┤
                    └─ reasoning ──────→ Claude
```

Predictable generation has a similar route:

existing code → cheap worker model → generated file written to disk

Claude doesn't have to take all of the generated code into its context.

## Context is becoming something we should budget

Until now, choosing a model has mostly been a question of quality: the smartest, the fastest or the cheapest. Agent systems make that framing less useful, because a single task mixes different kinds of work.

Say I ask an agent to look into a problem in a repository with twenty large files. That involves at least three operations:

1. get the relevant information
2. think about the problem
3. alter the correct code

There is no reason all three should use the same model. The first is mostly fact extraction. The second needs the best reasoning model. The third probably needs that model too, because small errors matter.

Using a frontier model for every stage treats intelligence as a flat resource. Some tokens go to decisions; many more go to moving information around.

## Spotify enforces the routing

The first version apparently put routing instructions in `CLAUDE.md`, so Claude would know when to hand work to the cheaper model. That was unreliable: the instructions were advisory, and Claude could ignore them.

The current implementation uses a Claude Code [plugin named `shunt`](https://github.com/spotify/portal-ai-plugins/blob/main/plugins/shunt/README.md), with hooks that intercept specific operations before they run.[^1] A large file read, for example, can be blocked and redirected to the bulk-reading worker:

Claude wants to read 2,000-line file → hook intercepts → bulk-reader model → concise result → Claude

Targeted reads still go straight to Claude. If the model already knows it needs lines 430 to 510, there is no reason to send that through another model.

The difference between **advisory routing** and **enforced routing** matters. I expect many agent systems where the harness, and not the model, decides which resources an operation gets. It is the same architectural shift I wrote about with GPT-6 Astra two days ago.

## The 90% number has important limitations

Spotify applied this to a Java monorepo and reports average savings of about 90% of Claude-side tokens for bulk reading.[^1] That does not mean Claude Code in general gets 90% cheaper.

The worker model is deliberately kept out of some work. Spotify found it caught surface-level code patterns but missed a subtle thread-safety issue that Claude found once it had the right context. Editing is also hard, because the summaries don't reliably keep the location information needed for exact changes.[^1]

There is a latency cost too. A delegated operation is an extra model call and usually takes 10 to 30 seconds, so routing a small file through the worker is slower than letting Claude read it directly.

So the problem is not:

> use cheaper model

It is:

> decide which work deserves the expensive model

## Agents may start looking more like operating systems

When different models are good at different tasks, the agent harness starts to look like a scheduler. It knows which resources are available, estimates what a task needs, and decides where the task runs.

We may end up describing models less like applications and more like compute classes:

```text
fast / cheap model
    → extraction
    → classification
    → bulk reading
    → predictable generation

frontier model
    → debugging
    → architecture
    → ambiguous decisions
    → difficult edits

specialized model
    → vision
    → security analysis
    → formal reasoning
```

The top-level agent decides how they combine. That is one reason I am increasingly skeptical of products whose most important setting is a model dropdown. An agent system could use several models within one task without the user knowing which one handled each intermediate step.

## The scarce resource may be frontier-model attention

Human teams already work this way. The most experienced engineer on a team doesn't usually go through 40 log files by hand and copy out the relevant lines before looking into an incident. Other systems or people gather the information, so the expensive expertise goes to the part of the problem where it makes the most difference.

We treated frontier models differently because model calls first seemed cheap. Agent workloads change that. A coding agent working over a long session can process millions of tokens: reading repositories, re-reading its own output, repeating tool results and carrying state between actions. When agents run for hours without a new prompt, wasting the most capable model's context becomes a real cost.

The optimisation is to treat frontier-model attention as scarce: compress information with cheap systems, and spend expensive intelligence where a real decision has to be made. Spotify's is an early implementation of that idea.

---

### References

[^1]: [**Spotify Engineering, “Portal by Spotify cut my Claude Code token usage by 90%,” 3 September 2026.**](https://engineering.atspotify.com/2026/9/portal-by-spotify-cut-my-claude-code-token-usage-by-90) Dimitri Mazmanov explains that he used Portal's AiKA Modes together with a Claude Code plugin named [`shunt`](https://github.com/spotify/portal-ai-plugins/blob/main/plugins/shunt/README.md) in order to send bulk file reading and the generation of predictable code to worker models. The article states that there was an average savings of about 90% of Claude tokens during bulk-reading tests in a Java monorepo, although it also notes the limitations concerning editing, reasoning quality and extra latency.
