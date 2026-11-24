---
title: "Will engineers forget systems that AI keeps fixing?"
url: "https://bosnadev.com/2026/09/05/would-engineers-forget-how-their-systems-work"
author: "Mirza Pašić"
date: "2026-09-05"
topic: "Practice"
tags: ["sre", "incident-response", "automation", "skills", "comprehension-debt"]
summary: "Routine incidents teach engineers how their systems fail. If AI handles them, people may reach the hardest failures with less practice, and averages such as MTTR may hide that loss."
---

# Will engineers forget systems that AI keeps fixing?

An AI incident responder can manage alerts without needing to wake anyone. It can analyze telemetry and recent deployments, determine what likely failed, and even sometimes apply a fix. However, [Sylvain Kalache](https://www.sylvainkalache.com/blog/ai-handles-incidents-engineers-lose-touch-with-their-systems) is concerned about what engineers might stop learning if this becomes the norm.[^1]

At three in the morning, I would prefer to let the AI handle a routine capacity issue. Waking an engineer to repeat a known fix offers little benefit. The service recovers more quickly, and someone gets to sleep.

Routine incidents are crucial for engineers to learn how a system reacts when something goes wrong. If the AI addresses these cases, a human may only be called in after the automation has failed. Their next incident will start from where the automation stopped, leaving them with less experience handling simpler failures.

This highlights an ongoing issue with automation in the context of incident response.

## Automation leaves the most challenging tasks for people

In her 1983 paper, ["Ironies of Automation"](https://doi.org/10.1016/0005-1098%2883%2990046-8), human factors researcher Lisanne Bainbridge addressed this issue. Although she was specifically discussing industrial control systems, her insights apply to anyone who has ever been on call.

Automation typically takes over routine tasks because humans struggle to perform them consistently. As a result, operators monitor systems that seldom require intervention and step in only when something unusual occurs. Improved automation reduces the opportunities for operators to practice their skills, even though they still need to be competent enough to manage issues that arise beyond the machine's capabilities.[^2]

Consider an engineer who previously dealt with a variety of incidents, including:

- Routine capacity problems
- Misconfigured deployments
- Database connection exhaustion
- Expired certificates
- Bad feature flags
- Unusual latency spikes
- Partial regional failures
- Unknown cascading failures

An automated system that addresses the first five issues significantly eases the engineer's workload. However, this leaves the following challenges:

- Unusual latency spikes
- Partial regional failures
- Unknown cascading failures

While the engineer now handles fewer incidents, the remaining issues tend to be more complex. A year-end reliability report may show fewer pages of incidents and faster resolutions. However, it is unlikely to indicate whether team members truly understand the underlying causes of the failures without assistance from the automation.

Imagine that the first year produces these numbers:

| Metric                                                         | Change |
| -------------------------------------------------------------- | ------ |
| pages requiring a human                                        | ↓ 70%  |
| average MTTR                                                   | ↓ 45%  |
| after-hours interruptions                                      | ↓ 60%  |
| automatically resolved                                         | ↑ 80%  |
| engineers who can explain the failure without asking the agent | ↓ 50%  |

The figures presented are hypothetical. The first four would likely be included in a quarterly engineering review, while the fifth would require someone to take measurements.

## Routine incidents teach the system

Much of my understanding of systems comes from observing their failures in confusing ways. Architecture diagrams provide some clarity, but incidents reveal which dashboards are useful, which metrics could mislead me, and which dependencies to investigate when a graph changes unexpectedly.

You cannot fully absorb this knowledge from documentation alone. A person who has diagnosed ten common database issues is likely to approach the eleventh, more unusual problem with better intuition than someone who has only read about these issues. The specific commands matter less than the mental model developed while sifting through incomplete evidence.

This gap in understanding is often referred to as **comprehension debt**: the difference between how much software a team manages and how well they understand it. Recent research has explored this issue in AI-assisted development, noting that people can generate and deploy code without developing a useful understanding of its functionality.[^3]

## Incident automation can widen this gap

AI can produce software before a team fully understands it, and it can operate that software while it is functioning properly. However, the owners may have little direct interaction with the system until an unfamiliar failure occurs.

## Average MTTR can conceal failures

[Kalache](https://www.sylvainkalache.com/blog/ai-handles-incidents-engineers-lose-touch-with-their-systems) predicts that AI assistance could reduce recovery times for most incidents while making the rare human escalations slower.[^1] Relying solely on averages may obscure this outcome.

For example, consider a team currently managing 100 incidents:

| Incidents             | Time            |
| --------------------- | --------------- |
| 95 routine incidents  | 30 minutes each |
| 5 difficult incidents | 4 hours each    |

After it introduces an incident agent:

| Incidents             | Time           |
| --------------------- | -------------- |
| 95 routine incidents  | 5 minutes each |
| 5 difficult incidents | 7 hours each   |

I created those numbers to illustrate the distribution of incident response. Overall, the Mean Time to Repair (MTTR) improves, even as the most serious incidents take longer to resolve.

Teams that adopt AI Site Reliability Engineering (SRE) tools should measure several key metrics: the number of incidents the AI agent resolves, the number of incidents it escalates immediately, and those incidents it investigates before handing over to a human engineer. The latter is important because the engineer inherits both the original failure and the agent's unfinished investigation.

The handover process itself also needs to be measured. After twenty minutes of autonomous investigation, can an engineer understand what the agent has discovered? Providing a concise timeline, verified telemetry, and discarded hypotheses may help. In contrast, being presented with hundreds of tool calls and five competing theories may leave the engineer in a worse position than when the investigation began.

It’s important to note that reading an incident report is different from diagnosing the incident itself. Reports outline the symptoms, hypotheses, and solutions after someone has resolved most uncertainties. During a live failure, an engineer must sift through incomplete and contradictory information to find a signal.

This distinction also exists in programming. A clean solution to a complex bug can seem obvious after the fact. However, identifying it while multiple plausible causes, such as issues with the database, network, application, deployment, cache, or another team's service, are present requires a different skill set.

Practice must occur in some form. Modern aircraft automate most normal flight operations, but pilots remain accountable during rare abnormal situations. The aviation industry does not wait for a real engine failure to conduct practice. Pilots rehearse abnormal and emergency procedures using simulators as part of their recurrent training.[^4]

Software teams already have similar tools at their disposal. Game days, disaster recovery exercises, tabletop simulations, chaos engineering, and deliberately broken staging environments enable engineers to study failures without impacting production. Improved observability, runbooks, and automation should continue to make production easier to manage, while these exercises help retain valuable experience that routine tasks might overlook.

Organizations like [Rootly](https://rootly.com/) and [Uptime Labs](https://uptimelabs.io/) use simulated outages for training exercises.[^1] During these exercises, an engineer acts as the incident commander while Large Language Model (LLM)-driven participants simulate the roles of individuals involved in the response. The engineer must diagnose the outage using incomplete telemetry and coordinate the response.

I recommend starting with incidents the team has already experienced. Change one symptom, keep the AI responder out of the exercise, and ask an engineer to diagnose the incident before revealing the agent’s analysis. The agent can take on various roles as needed, such as a customer reporting symptoms or a dependency owner answering questions.

Creating these exercises requires preparation, especially if built manually. However, AI can assist by generating telemetry, role-playing participants, and adapting scenarios based on the responder's decisions. The engineer still has to navigate uncertainty.

We already test backups by restoring them. Similarly, if senior engineers are the final step in the recovery plan, their ability to take over should be assessed as well.

The same issue arises in the development process. Coding agents can now explore unfamiliar codebases, follow call graphs, find configurations, generate migrations, update dependencies, write tests, and diagnose failures, all sensible tasks to delegate. However, these tasks were also common ways for developers to learn about a codebase.

Code reviews can also create additional separation. One agent writes an implementation, and another reviews it. This approach increases throughput, allowing the engineer responsible for the service to approve changes without deeply understanding their construction.

I choose to use coding agents because they speed up development. However, I must decide which tasks I should delegate based on whether I need just the results or also the understanding that comes from executing the work.

A repetitive serializer is mostly about the resulting code. However, investigating why a distributed system behaves differently under load enhances my understanding of that system. Delegating both tasks in the same way can eliminate the repetitions that prepare someone for a production failure.

It is crucial to determine which skills can become obsolete. Every useful abstraction diminishes the necessity of some older skills. For example, most web developers can’t implement [TCP](https://www.rfc-editor.org/rfc/rfc9293.html) from scratch. Most application developers don’t grasp exactly how their database writes pages to disk, and few engineers managing [Kubernetes](https://kubernetes.io/) could rebuild it. Software exists in layers because no one can comprehend all of them.

AI will accelerate this process. Some knowledge that appears essential today may soon become just another implementation detail beneath the layer where people operate. The decision on which skills to maintain depends on where the safety boundary lies.

An organization may eventually trust automated systems to recover from each other's failures. In such a design, extensive human operational knowledge may become less critical. Many current systems still operate within these boundaries.

AI cannot resolve incident → page senior engineer

In that system, the senior engineer's knowledge is a crucial part of the recovery process. Bainbridge's observation highlights that while automation eliminates routine tasks, it leaves the most challenging cases for the individuals who have had the least recent experience with them.[^2]

AI incident response can significantly reduce the amount of repetitive on-call work for team members. Teams that maintain a human as the final point of escalation should dedicate some of the time saved to conducting simulations and handover tests. Additionally, they should track and assess the difficult incidents separately from the routine ones.

---

### References

[^1]: [Sylvain Kalache, “AI handles incidents, engineers lose touch with their systems,” 4 September 2026.](https://www.sylvainkalache.com/blog/ai-handles-incidents-engineers-lose-touch-with-their-systems) Kalache argues that AI responders can reduce routine operational work while removing the practice through which engineers build operational intuition. He discusses comprehension debt and incident simulation as a way to maintain those skills.

[^2]: [Lisanne Bainbridge, “Ironies of Automation,” _Automatica_, 1983.](https://doi.org/10.1016/0005-1098%2883%2990046-8) Bainbridge describes the problem created when automation handles routine operations but leaves people responsible for abnormal situations, reducing their opportunities to maintain the skills those situations require.

[^3]: [Muhammad Ovais Ahmad, “Comprehension Debt in GenAI-Assisted Software Engineering Projects,” April 2026.](https://arxiv.org/abs/2604.13277) The study uses 621 reflective diaries from 207 students. It defines comprehension debt as the gap between what a team knows about its codebase and what it needs to know to maintain and modify it effectively.

[^4]: [FAA / CIAIAC accident-report material on recurrent training and proficiency checks.](https://www.faa.gov/sites/faa.gov/files/2022-11/Final%20CIAIAC%20Rpt%20Spanair%205022.pdf) The report includes engine-failure situations in recurrent flight training and proficiency checks conducted every six months, one example of aviation deliberately rehearsing rare abnormal events.
