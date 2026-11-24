---
title: "Elsewhere today: Nvidia buys Hugging Face, Qwen 3.8"
url: "https://bosnadev.com/2026/09/03/elsewhere-today"
author: "Mirza Pašić"
date: "2026-09-03"
topic: "Links"
tags: ["roundup"]
summary: "Qwen 3.8 at 1,500 tokens a second, an open model with checkpoints and logs, coding agents agreeing 42% of the time, and Nvidia buying Hugging Face."
---

# Elsewhere today: Nvidia buys Hugging Face, Qwen 3.8

**[Qwen 3.8 27B is now running on Cerebras](https://inference-docs.cerebras.ai/models/overview).** The figure quoted on Hacker News was about 1,500 tokens per second. Benchmarks aside, inference at that speed changes how an LLM feels as a product: long agent loops, retries and tool calls get much less frustrating once the model stops being the bottleneck.

[**K2 Horizon**](https://ifm.ai/blog/k2/) is an unusually serious open-model release. IFM released six models from 0.9B to 375B parameters, and what I appreciate most is what comes with them: intermediate checkpoints, training code, configurations, logs, and either the training data or detailed instructions for building it. That is much closer to what "open" should mean than uploading weights to [Hugging Face](https://huggingface.co/).

[**Which third-party services does Claude Code, Codex and Cursor select?**](https://armature.tech/blog/which-tools-coding-agents-install) Armature ran 16,893 sessions across 75 repositories to see which third-party services coding agents actually pick. The three agents made the same choice only 42% of the time. If you build developer tools, the one choosing between three APIs is often the coding agent that runs `npm install`, and not the developer.

**[Audacity 4.0 has just been released](https://www.audacityteam.org/changelog/).** A major update: the interface is rewritten in Qt, clip editing is redesigned, workspaces are new, Windows gets official ASIO support, and projects save in the new `.aup4` format. I have a soft spot for releases where an established open-source program gets an ambitious redesign instead of slowly turning into abandonware.

**[Polars 2.0 is in pre-release](https://pola.rs/posts/announcing-polars-2/).** Polars has become one of the more interesting parts of the current data stack, especially if you prefer Rust-backed tools to reaching for pandas by default. A major version is worth watching even if you don't use it yet.

[**Xanadu was waiting for agents**](https://zed.dev/blog/agentic-xanadu) is a well-written piece from Zed connecting Ted Nelson's ideas about hypertext to coding agents. Agents show how primitive our relationship with source code still is: files and line numbers are convenient storage primitives, but they are not necessarily the best way to represent the relationships an agent needs to follow across a large codebase.

Nolan Lawson's [**The asteroid currently hitting frontend web development**](https://nolanlawson.com/2026/08/23/the-asteroid-currently-hitting-frontend-web-development/) was widely shared today. The asteroid is AI coding. I don't accept every conclusion he draws from that, but the general observation is hard to ignore: AI is changing how frontend code is written, and also which abstractions, libraries and documentation styles survive once agents are major users of them.

[**Nvidia is acquiring Hugging Face for $12.93 billion**](https://blogs.nvidia.com/blog/nvidia-to-acquire-hugging-face/). Hugging Face claims more than 18 million developers and researchers, three million models, 500,000 datasets and one million applications. Nvidia says the platform will stay neutral on models, frameworks, clouds and hardware. This deserves more than a short mention: owning the leading AI computing platform and one of the main distribution channels for open models is an exceptional combination.
