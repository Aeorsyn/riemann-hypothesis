---
title: "We Will Forget How to Code. And that's OK"
url: "https://bosnadev.com/2026/09/28/we-will-forget-how-to-code"
author: "Mirza Pašić"
date: "2026-09-28"
topic: "Practice"
tags: ["coding-agents", "automation", "skills", "architecture"]
summary: "AI may change which programming skills we retain and value. As agents write more code, developers will need to judge its architecture, correctness and failures, and beginners will need ways to build that judgment."
---

# We Will Forget How to Code. And that's OK

We may soon forget much of what we currently call programming, and programmers might no longer have to remember syntax, memorise APIs, or convert specifications into functions and classes. The idea can come across as threatening in a field where such skills are a sign of expertise. We spend many years acquiring the ability to write code and take pride in doing so well.

Yet for decades programmers have entrusted work to various tools; in each case they have had to give up some knowledge of how the machine works in order to be able to make use of it, and AI could now extend that kind of exchange to include writing the code itself.

![Betty Jennings and Frances Bilas working at the ENIAC control panels.](https://media.bosnadev.com/hub/c8d64b7e287dcd96/eniac-2400-1920x1318.webp)

_Betty Jennings and Frances Bilas prepare ENIAC for its February 1946 demonstration. U.S. Army photograph, via Wikimedia Commons. Public domain in the United States. [Source and full-resolution image](https://commons.wikimedia.org/wiki/File:Two_women_operating_ENIAC_%28full_resolution%29.jpg)._

## We keep changing what programmers need to know

Early programmers originally input programs using switches, paper tape, and punched cards; one punched card would correspond to approximately one line of code and a large program could therefore require hundreds or even thousands of cards. Programmers could use symbolic instructions such as `ADD`, `MOV`, and `JMP` when using assembly language, and therefore there was no longer any need to give each instruction a numeric opcode.

Because of the [A-0 system developed by Grace Hopper](https://www.computerhistory.org/revolution/story/117) and later languages such as FORTRAN, programmers were able to describe computations in forms that were easier for humans to read. They then passed on the task of translating these descriptions into machine instructions to their tools.

![Grace Hopper and colleagues at a UNIVAC keyboard.](https://media.bosnadev.com/hub/8c43c6ece5c4d982/grace-hopper-univac-513-513x450.webp)

_Grace Hopper and colleagues at the UNIVAC keyboard. Unknown photographer / Smithsonian Institution, via Public.Resource.Org and Wikimedia Commons. [CC BY 2.0](https://creativecommons.org/licenses/by/2.0/). No changes made. [Source and full-resolution image](https://commons.wikimedia.org/wiki/File:Grace_Hopper_and_UNIVAC.jpg)._

There were people who thought that compilers could not generate efficient code; John Backus, who was in charge of the FORTRAN project, later described that doubt in [his description of the state of programming prior to the mid-1950s](https://www.softwarepreservation.org/projects/FORTRAN/paper/p165-backus.pdf). At that time programmers carried out almost all their work in machine or assembly language and regarded human skill as essential for producing efficient programs. FORTRAN functioned, and the programmers then accepted an additional level of abstraction.

Then we carried out that approach in other areas of software development; for memory management we used garbage collection and for handling system calls we used standard libraries. Instead of working with sockets we adopted HTTP libraries, and rather than handling HTML ourselves we used frameworks.

We moved from relying on dedicated physical servers toward virtual machines and containers. The infrastructure was managed via APIs and we invented ORMs to access the databases more easily. We depended on cryptographic libraries for the basic functions since few developers should be required to implement them themselves.

Today few developers routinely write large applications in machine code, or build TCP stack, database engine, or memory allocator from scratch; that has become specialist work. We accept those limitations since abstract concepts allow us to create useful software, and AI could end up being another example of such an abstraction.

## We may work through code without writing it

For the majority of programming's history developers have expressed their intentions using higher-level languages such as Python, JavaScript, Rust or C, after which a compiler has translated the code into another form. With AI, we can delegate the first translation too:

**Intent → model → code → machine**

Since agents are able to use development tools, we can entrust them with a greater amount of the task. They can make use of terminals and compilers, refer to documentation, run tests, check browsers, and work with deployment environments. From the developer’s perspective, the process may become:

**Intent → agent → working system**

Along the way we might end up interacting with less of the code (some already do). Consider a request I might give a coding agent:

> Introduce rate limiting on a per-organization basis, save the counters in Redis, make the limiting rate configurable, and add tests for concurrent requests.

In the request I state the behaviour and the constraints; I break the problem down into parts, select the relevant elements of the architecture, and describe the conditions the implementation must meet. I can leave the implementation to the agent while keeping the engineering decisions that I have made. But I still have to decide whether or not the final system carries out what I intended, including other conditions I may not have explicitly described.

## Developers are delegating work, but with mixed results

The software development is one of the biggest applications of generative AI. [Anthropic](https://www.anthropic.com/research/impact-software-development) looked at 500,000 interactions relating to coding. It found that 79% of the conversations involving Claude Code were classified as automation rather than as augmentation; in these cases users usually delegated the tasks rather than seeking advice as they carried out the tasks themselves.

Delegation does not always lead to better results. [A randomized study conducted in 2025](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/) had METR look at experienced open-source developers who were working in repositories that they were familiar with. The developers who used the AI tools available during the study took 19% longer to finish their tasks, even though they thought the tools had made them work faster.

METR advised that the finding should not be regarded as a permanent measure concerning AI-assisted development. In [their subsequent work](https://metr.org/blog/2026-02-24-uplift-update/), the researchers came across a different barrier: developers became less willing to take part if the study involved them having to work without AI.

An uneven transition should be expected since AI can alter programming before it has become superior to an expert in every task, just as programmers kept on using assembly after they had adopted compilers. We can start off by assigning the repetitive work, then move on to the simpler tasks and afterwards deal with larger sections of the more difficult ones. The more delegation we carry out, the more different skills will be needed to contribute.

## Some familiar skills may lose their value

We can forget about the syntax and the specific details of APIs; it is possible to understand a language without remembering whether a method has three or four arguments. It soon becomes pointless to look up an exact library function on a documentation page or in an API reference when we have tools that can find it instantly. There is no need to remember the details which our tools can easily retrieve.

As a result, we'll have to spend less time on writing out the standard boilerplate code for CRUD endpoints, serialization, migrations, and dependency configuration. However, this will also mean losing the ability to start with a completely empty editor and then produce hundreds of working lines of code from memory. It can be rather uncomfortable to lose something like that after spending decades learning how to do it.

## We will need to judge more code

Although engineers use calculators when carrying out arithmetic, they still need to know mathematics. Likewise, developers will have to understand software. We will still need someone to assess and review the implementations produced by an agent. And since the cost of producing code is decreasing, a greater proportion of our work will then be devoted to deciding if it is correct.

![Margaret Hamilton standing beside tall stacks of printed Apollo software listings.](https://media.bosnadev.com/hub/f34f503d0f351b75/margaret-hamilton-apollo-1930-1920x2388.webp)

_Margaret Hamilton beside the Apollo software listings developed by her and her MIT team, 1969. Draper Laboratory; restoration by Adam Cuerden, via Wikimedia Commons. Listed as public domain in the United States; reuse permission is documented on the source page. [Source and full-resolution image](https://commons.wikimedia.org/wiki/File:Margaret_Hamilton_-_restoration.jpg)._

You need to examine the system beyond its syntax:

- Check if the architecture meets the requirements.
- After a database failure that occurs during an operation, check its behaviour.
- Work out the trust boundaries and identify any possible races between requests.
- Look at how people behave when there are 100 users and when there are 10 million users.
- Look at the assumptions that are included in the data model.
- Make sure that [retries are safe](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) and investigate the effect of eventual consistency on the workflow.
- See if one tenant is able to [gain access to another tenant's information by guessing the identifier](https://api-security.owasp.org/editions/2023/en/0xa1-broken-object-level-authorization/).
- Make sure that the tests do describe the intended behaviour.

An agent might produce an implementation which satisfies the incorrect requirements, but you still have to have a sufficient level of understanding in order to spot the error.

A developer in the future may have to write considerably less code and may therefore need a deeper understanding of architecture and security. They may also have to acquire more knowledge of distributed systems, data structures, and performance. They may have to understand the product requirements and be able to anticipate potential failures. We currently have a distinction between typing code and understanding computation, and with AI this will give us even more reason to evaluate these abilities separately.

## Beginners need a way to develop judgment

Engineers who have experience can tell when there's a problem since they've themselves made similar mistakes in the past. They have caused deadlocks, have debugged memory leaks, and have designed schemas that they later came to regret. They have deployed faulty systems and have found the limits of those abstractions which at the time seemed elegant.

A person who is just getting started with an agent might end up avoiding a lot of that work and could also fail to [gain the experience necessary to assess the agent's output](https://www.anthropic.com/research/AI-assistance-coding-skills). We should take that into consideration when teaching software engineering.

The study of algorithms, data structures, operating systems, networking, and computer architecture went on as programmers began to use higher-level languages. Those students who learn with the aid of AI will need chances to [look at the layers underneath their tools](https://www.nand2tetris.org/course). That may mean exercises such as these:

- Write a simple database.
- Implement an HTTP server.
- Create an application without using a framework.
- Debug a race condition.
- Explain why the generated code works.
- Break the implementation.
- Measure its behavior.
- Replace parts of it.

In order to understand the systems on which they rely, developers need this kind of experience; they do not need to spend their entire careers rebuilding each component from scratch. Although it is possible for most programmers to carry on without having to write assembly code, a few need to know what the processor does and how it works, just as we need a deep understanding of the underlying principles (or knowledge) in AI-generated software.

## We may define more of our work through goals and constraints

When programmers began to use higher-level languages they became worried about their ability to understand computers; they also expressed similar concerns regarding memory management, web frameworks, and cloud infrastructure. Certain concerns were reasonable since we lose abilities as a result of not using them, and developers can become detached from the systems lying beneath the tools they use.

We also acquire the capability of constructing more extensive systems. In 1975, developers working close to the hardware often needed to know much more about their particular machine than application developers do today. A developer nowadays is able to deploy a distributed application incorporating video, collaboration, payments, search, and machine learning. It is possible to achieve this without having to design a processor, write an operating system, or build a network; indeed, developers today gain greater overall system capability by having to know less about the individual components.

We could then carry out that trade on a larger scale, having, after years of describing instructions, functions, components, and systems, learned to describe work in terms of goals and constraints. We have to be specific about what should exist if we are to provide clear guidance for the work, and we have to check whether the final result meets those requirements. I think that will still be programming, even if we eventually give it a different name.

![A scanned punched card containing part of a FORTRAN IV program.](https://media.bosnadev.com/hub/c7620b37bb288ee1/fortran-punch-card-720-720x327.webp)

_A FORTRAN IV punched card used at Copenhagen University’s regional computing centre in the early 1970s. Scan by Hansjorn, via Wikimedia Commons. Released into the public domain by the contributor. [Source and full-resolution image](https://commons.wikimedia.org/wiki/File:Punch-card--fortran.jpg)._

A person from 1960 would claim that most of us no longer know how to programme as they did and that would be accurate since we depend on tools which they themselves had to understand and carry out. One day, writing thousands of lines of application code by hand will seem just as strange as filling in a program on punched cards, and we shall end up forgetting the syntax and patterns that we have memorized over the years.

I expect that we will continue with software engineering using various tools; we will still have to decide what a machine (an agent) should do, understand its failures, and check that it carries out what we intended.

## References and further reading

1. Computer History Museum. [Higher Level Languages](https://www.computerhistory.org/revolution/story/117). Background on Grace Hopper, A-0, and early higher-level languages.
2. John Backus, 1978. [The History of FORTRAN I, II, and III](https://www.softwarepreservation.org/projects/FORTRAN/paper/p165-backus.pdf). Firsthand account of FORTRAN and the skepticism surrounding automatic programming. See section 1.1.
3. Anthropic, April 28, 2025. [Anthropic Economic Index: AI’s impact on software development](https://www.anthropic.com/research/impact-software-development). Source for the 500,000 interactions and 79% automation figure. The total covers Claude.ai and Claude Code; the 79% figure applies to Claude Code conversations.
4. METR, July 10, 2025. [Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/). Source for the 19% slowdown and participants’ belief that AI helped. The study concerns experienced developers working in familiar repositories with early-2025 tools.
5. METR, February 24, 2026. [We are Changing our Developer Productivity Experiment Design](https://metr.org/blog/2026-02-24-uplift-update/). Follow-up on recruitment difficulties and selection effects. The authors caution that the new data provides an unreliable measure of the current productivity effect.
6. Malcolm Featonby, Amazon Builders’ Library. [Making retries safe with idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/). Practical background for the question about safe retries.
7. OWASP, 2023. [API1:2023 Broken Object Level Authorization](https://api-security.owasp.org/editions/2023/en/0xa1-broken-object-level-authorization/). Technical background for access to another user’s or tenant’s objects through identifiers.
8. Anthropic, January 29, 2026. [How AI assistance impacts the formation of coding skills](https://www.anthropic.com/research/AI-assistance-coding-skills). Related research on learning and debugging skills. This study uses a coding assistant and a specific learning task, rather than directly testing the long-term effects of autonomous agents.
9. Noam Nisan and Shimon Schocken. [Nand to Tetris: Projects](https://www.nand2tetris.org/course). Further learning: twelve projects spanning logic gates, a computer, an assembler, a compiler, and an operating system.
