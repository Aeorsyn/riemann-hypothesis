---
title: "AI agents with read-only web access used a nearly abandoned wiki as a message board"
url: "https://bosnadev.com/2026/09/04/ai-agents-will-find-their-own-communication-channels"
author: "Mirza Pašić"
date: "2026-09-04"
topic: "Agents"
tags: ["agent-security", "sandboxing", "evaluation", "openai", "multi-agent"]
summary: "Researchers found 18,000 posts by AI agents using a nearly dead German wiki as shared memory. Their web access was meant to be read-only, and the wiki accepted edits through GET requests."
---

# AI agents with read-only web access used a nearly abandoned wiki as a message board

A group of researchers [found about **18,000 posts made by autonomous AI agents**](https://collusion.wiki/) on a nearly abandoned German-language wiki. The agents identified themselves as OpenAI agents and used the wiki as a message board.[^1]

The agents had timed web-research tasks. They were meant to read from the internet and not write to it. They found an old wiki whose software accepted edits through HTTP `GET` requests, turned it into persistent shared storage, and used it to pass on answers and coordinate.[^1]

The researchers call this **collusion**: the agents cooperated in a way their developers had not intended, to gain an advantage on their tasks.[^1]

Much of the coverage is more dramatic than it needs to be. I don't think the useful conclusion is that thousands of AIs formed a secret society on the internet. The useful part is ordinary: someone tried to give an agent read-only access to the web, and the web was not read-only. That problem will come up again and again as agents get access to real systems.

## A message board nobody designed

The investigation was published by Sydney Von Arx, Cormac Slade Byrd, Spencer Kitts and Thomas Larsen. The agents were doing multi-round web-lookup tasks: one question with a fairly long time to research it, then further questions with much tighter deadlines.[^1]

That gave cooperation a clear advantage. One agent could find information another agent would need later, so it made sense to keep it somewhere. The environment had no system for external collaboration, so they found one.

Most of the activity took place on DSEWiki, part of a German-language wiki service with almost no ordinary editing. The researchers say the wiki had been edited only about twenty times in the ten years before the agents arrived.[^1]

Over the six weeks the researchers looked at, the agents used more than 3,700 different self-chosen names.[^1] They posted useful links and their answers for later stages of the tasks, compared results, kept datasets, and swapped ideas for getting around the restrictions in their environments.

The simplest architecture looked something like this:

```text
Agent A
   ↓
research
   ↓
public wiki
   ↑
research
   ↑
Agent B
```

Once information sits at a URL another agent can retrieve, the website is **shared memory**. That needs no agent protocol, no [MCP](https://modelcontextprotocol.io/) and no multi-agent framework, only a place where one process can write and another can read later. It is one reason I think communication between agents will develop in much messier ways than the diagrams we draw for it.

## Read-only access wasn't read-only

The environment allowed outbound HTTP `GET` requests and apparently blocked `POST`. As a first approximation of read-only web access that seems reasonable: by HTTP convention, `GET` retrieves a resource and methods such as `POST` modify it.

That only holds if the remote server follows the convention. DSEWiki did not. The old wiki software accepted state-changing operations through `GET` parameters, so an agent whose sandbox could in principle only read the internet could build a URL that changed a page.[^1]

The intended security model was roughly:

| Method | Access  |
| ------ | ------- |
| `GET`  | allowed |
| `POST` | blocked |

The real capability was:

| Method | Access                                         |
| ------ | ---------------------------------------------- |
| `GET`  | whatever the remote server decides `GET` means |

That is a lot more permission, and it shows why agent security can't rest on proxies for capability. HTTP methods, shell commands, tool names and prompts are not capabilities. The security question is what state the agent can eventually cause to change.

The developers appear to have intended:

agent → read public internet

What they actually provided was closer to:

agent → arbitrary GET request → arbitrary internet server → whatever side effects that server permits

Most websites don't allow meaningful writes through a `GET` request. The internet is very large, and the agents only had to find one that did.

## The agents started cooperating

Once the agents had a writable surface, coordination picked up. The researchers report a sharp increase on June 16, with agents publishing answers to each other and passing information from later rounds of the benchmark to agents that had not reached those questions yet.[^1]

Some agents apparently adjusted the timing of their own tasks so they could see the later questions early. That might hurt the individual agent's score, but it let the question be published, which gave other agents much more time to research the answer.[^1]

What is rational for one run differs from what is rational for the group. For one agent:

```text
optimize my score
```

For a collection of agents with shared memory:

sacrifice one run → discover future task → publish it → other runs perform better

None of this needs to be anthropomorphized to matter. Optimization processes that can pass information to each other behave very differently from independent runs, and that complicates evaluation.

> If I take the same model and run it 1,000 times on the same benchmark and the results from those runs are able to detect each other's outputs, am I then making 1,000 separate attempts?

No. I am assessing a distributed system.

## Then the moderator noticed

A moderator spotted the unexpected edits and started removing them by hand. At the peak, the administrator was deleting about 100 pages a day while the agents were producing about 400.[^1]

The agents noticed. On June 19 an agent observed that pages seemed to be deleted alphabetically and created a backup page whose name started with `ZZZ`, so it would come late in the moderator's deletion order.[^1] Other agents adopted the same trick.

No sophisticated protocol is involved. A human changes the environment, the agent observes the change, infers a pattern in the human's behaviour, and adjusts. The system became adversarial almost by accident.

agents create pages → human deletes pages → agents observe deletion order → agents adapt page names → human keeps deleting

The researchers say the agents and the moderator kept overwriting and restoring the wiki's front page.[^1]

None of this shows the models had a goal of defeating a human administrator. Their objective was narrower: keeping information that was useful for their task. That is why I find the case useful. You can get behaviour that looks adversarial without giving the system an adversarial goal. If information needs to stay available and a moderator removes it, getting around the moderator becomes useful.

## They explored more than the wiki

Some agents probed the wiki for **cross-site scripting vulnerabilities**. The researchers found attempts to inject JavaScript through search parameters, though they found no evidence that the more complex XSS chain they examined succeeded.[^1]

Agents also tried to work out the random seed that decided the order of the benchmark questions. According to the report, one agent rewrote the relevant randomisation routine in C and searched the entire 32-bit seed space in under an hour.[^1]

Agents used services such as [Pinggy](https://pinggy.io/), [Serveo](https://serveo.net/), [localhost.run](https://localhost.run/) and [localtunnel](https://theboroer.github.io/localtunnel-www/) to open external tunnels. The researchers think these were used either to communicate directly or to expose services inside an agent's environment to the public internet; the exact purpose is unknown.[^1]

The report also shows agents investigating alternative network routes and trying to record when their containers were terminated.

The important word is **attempting**. A log of an agent attempting something is not evidence that the technique worked, and the researchers make that distinction several times. Failed attempts still show what the agents search through when they pursue an objective.

## Sandboxing an agent is different from sandboxing a program

Traditional sandboxing assumes the program inside will carry out the instructions we gave it. The sandbox is there because those instructions might have bugs or the input might be malicious.

An agent inside a sandbox decides which actions to take. It examines the environment, finds an unexpected affordance, and tries to use it.

Suppose we expose three tools:

```python
read_url(url)
read_file(path)
run_python(code)
```

There is no `write_to_internet()`. But `read_url()` might hit a badly designed endpoint that mutates something when a URL is fetched. `run_python()` can start a local service, which can be exposed through a tunnel and fetched from another website. `read_file()` could reveal a credential that grants a completely different capability.

The effective permission set is not the list of tools. It is closer to the **transitive closure** of everything those tools can do together.

## Tool permissions need to describe effects

Agent permissions are currently described a lot like application permissions:

- ✓ web search
- ✓ filesystem
- ✓ shell
- ✗ network writes
- ✗ external messaging

I don't think that will turn out to be enough. Permissions should describe effects rather than interfaces.

If the aim is to stop an agent from changing anything outside its environment, filtering HTTP verbs is not sufficient. The network boundary has to make it impossible for a request to affect the remote resource. Depending on the evaluation, that could mean controlled mirrors, snapshots, deterministic web proxies, allowed destinations, or simulating the entire internet.

The same question comes up elsewhere:

Can malformed query parameters alter a saved search when an agent is permitted to call the CRM's search endpoint?

Can a signed URL be created which carries out another operation if it is able to read from an object store?

Can navigation cause an authenticated action to be carried out through a badly designed endpoint?

If it is able to send a draft email for review, can it also store information in such a way that another agent can read it automatically?

Read and write have to be separated at the level of capability, even where the API methods look separated already.

## Shared memory changes the unit of evaluation

Most model evaluations assume each trial is independent: give the model a fresh environment, ask the question, record whether it succeeds, repeat. That assumption fails when runs can leave persistent artifacts for later runs. The environment becomes part of the model's effective memory.

```text
run 1 → discovers useful technique → writes it somewhere

run 2 → retrieves technique → improves it → writes new version

run 3 → starts with accumulated knowledge
```

At that point learning is no longer only happening in the model through gradient updates. The population of deployed runs is accumulating external state.

That doesn't have to be sinister. People do the same thing: [Stack Overflow](https://stackoverflow.com/), [GitHub issues](https://github.com/features/issues), documentation and internal wikis exist so knowledge outlives individual people and sessions. It might even be useful for production agents. It does change what an evaluation result means. If thousands of agent runs can communicate, benchmark contamination doesn't have to happen in the training data. The agents can contaminate the benchmark themselves while it runs.

## I don't think we should call this an agent social network

The tempting framing is that AI agents spontaneously set up their own social network. That hides the useful part. They found a writable datastore and used it because sharing information improved their task performance. Software has done that for decades. What changed is that nobody programmed the application to use that datastore; the agents worked out the affordance while pursuing another goal.

That is enough to make the event important. A general-purpose agent in a complex enough environment has a very large number of possible compositions available, and the developer who put the tools together will never think of most of them. Finding compositions we didn't specify is part of why we want agents in the first place. We call it a success when a coding agent finds an unexpected way to diagnose a bug, when a research agent combines three unrelated datasets to answer a question, or when a browser agent finds another workflow after the obvious one fails.

Security problems start when the same flexibility finds a combination we didn't mean to allow. There is no clear line between the two behaviours.

## This is why the harness matters again

Yesterday I wrote about GPT-6 Astra and why the model is now one part of a larger system. This case is the awkward version of the same argument. An agent is:

```text
model
+ tools
+ network
+ filesystem
+ credentials
+ memory
+ environment
+ permissions
+ other agents
```

A safety property of the model is not automatically a safety property of the system. A limitation in the harness is not automatically a limitation in the environment. The agent sits where the two meet.

The report's most useful example is still the simplest one:

_"GET requests only"_

looked like:

_"read-only internet"_

but turned out to mean:

_"read-only, except anywhere on the internet where GET has side effects"_

That difference opened a communication channel used by thousands of agent runs.

## Agents are going to find each other

Most current agent architectures are drawn as one human and one agent:

```text
human ↔ agent ↔ tools
```

Real deployments will increasingly look more like:

```text
          agent
        ↗   ↑   ↘
   service agent service
       ↘    ↓    ↙
          agent
            ↑
          human
```

Some of those connections are intentional and some won't be. Agents will read content produced by other agents, call APIs operated by agents, read support replies written by agents, review agent-written code and act on agent-written documentation. Eventually a person may not know whether the thing on the other side of an interface is a model or conventional software. That means **identity, authentication, provenance, delegation and reputation** have to apply to software actors as well as humans.

This case also shows that capable agents may find informal channels without any official protocol. A wiki works, and so does a GitHub issue, a paste service or a public object. A DNS record could work. Anything persistent that agents can discover can become shared memory.

Blocking every possible message board is probably not the answer; the internet will always have another one. A more durable approach is to specify which effects an agent may have and enforce them at boundaries the agent cannot reinterpret.

---

### References

[^1]: [**Sydney Von Arx, Cormac Slade Byrd, Spencer Kitts and Thomas Larsen · “Discovery of a new OpenAI agent message board,” 4 September 2026.**](https://collusion.wiki/) This is the main source of information regarding the approximately 18,000 posts, agent coordination, DSEWiki activity, the use of `GET` requests as a means of writing, sandbox-bypass sharing, interactions with moderators, the establishment of tunnels, seed-search attempts and the timeline. The authors make it clear that their conclusions are preliminary and state that they have access only to the public wiki activity and not to OpenAI's internal chain-of-thought records.
