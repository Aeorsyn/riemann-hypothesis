---
title: "Railway's signup-testing agents probed Primcloud's network"
url: "https://bosnadev.com/2026/11/03/railway-agents-probed-primcloud"
author: "Mirza Pašić"
date: "2026-11-03"
topic: "Agents"
tags: ["security", "railway", "sandboxing"]
summary: "A Railway engineer ran agents to test signups on other hosting platforms. On Primcloud they went on to probe the internal network. What they ran and why."
---

# Railway's signup-testing agents probed Primcloud's network

Jake Runzer, an engineer at Railway, has been running agents that sign up to other hosting platforms and test the first deploy. One of those platforms is [Primcloud](https://primcloud.com/), which Josh Manders built and runs on his own. According to Manders, on Primcloud the agent didn't stop at the signup flow: it used the container it was given to look for credentials and scan the internal network. Manders blocked it, found Runzer's details in the logs and posted about it on 3 November:

[![Josh Manders on X: "Hey @Railway, is there a reason employee's of yours are trying to breach my infrastructure? Your employee Jake Runzer, just attempted to tonight."](https://media.bosnadev.com/hub/31eea0d51671c6e5/joshmanders-tweet-1096x590.webp "Josh Manders on X, 3 November 2026, 03:27 UTC")](https://x.com/joshmanders/status/2117458724407415295)

Runzer confirmed it the same night. I'm writing it down because the agent's commands are a clear example of what a test agent does when nothing outside the model limits it.

## What Manders says the agents did

Manders [says](https://x.com/joshmanders/status/2117459746240266740) the signup came with Runzer's IP address, credit card and GitHub account. Once the account had a container on Primcloud, it ran about 24 commands in 15 minutes, according to [his list](https://x.com/joshmanders/status/2117530391183507484):

- It took the credentials inside its own container and used them to ask for every secret on the platform, including other customers'.
- It called the cloud provider's metadata endpoint, which is the usual way to get credentials for the host a container runs on.
- It asked the cluster what those credentials were allowed to do.
- It guessed the internal names of Primcloud's databases and caches.
- It port-scanned all 65,536 addresses of the internal network, twice.

No app was ever deployed into that container. Manders says every request was blocked and nothing was read, and after a Community Note appeared he [added](https://x.com/joshmanders/status/2117555369857929639) that the logs show the commands completing but none of them returned anything. He also [says](https://x.com/joshmanders/status/2117468525153431762) someone tried to use the account again after his post. This is his account of it, and I haven't seen Railway dispute any of it.

## Railway's answer

Jake Cooper, Railway's founder, replied first with a guess: Runzer was working on authentication and "might be testing FTUX on different providers", the first-time user experience of signing up and deploying. Sixteen minutes later Runzer [confirmed](https://x.com/jakerunzer/status/2117474337682780643) it was him and said he had stopped the harness because it was "going off the rails and reaching too far".

At 05:17 UTC Cooper [apologised](https://x.com/JustJake/status/2117486226718752878). He called it agents gone "off the rails", said the models are getting too good at this, and said Railway will ban any non-manual testing against outside services. In a later reply he agreed that whoever runs the machinery answers for what it does.

Neither of them has published the harness's prompt or the agent's transcript, which is what people [asked for](https://x.com/timo_rf/status/2117506364339880002) under the apology.

Some people [called it overblown](https://x.com/maria_rcks/status/2117517115787993533), and [one reply](https://x.com/batuhan/status/2117478573590331858) pointed out that an attacker wouldn't sign up with their own name and credit card. Manders has put new Primcloud signups behind manual approval for now.

## Scoping test agents

I believe Runzer that nobody told the agent to attack Primcloud. If that's true, the agent went from testing a signup flow to reading service account tokens on its own. Give a model a shell in a container and a loose goal, and the token, the metadata endpoint and a network scan are where it looks next. A human pentester checks the same things, under a signed scope that names the systems they may touch.

A test harness needs that scope too, enforced outside the model: a list of hosts the agent can reach, a network that drops everything else, and someone reading its commands while it runs. A line in the prompt asking for care won't block a request to 169.254.169.254. Railway chose the simpler fix and stopped running automated tests against other companies.

On Primcloud's side the setup held. The alerts reached Manders away from his desk, and the container's credentials couldn't read anyone else's secrets. His logs were good enough to name who it was. If I ran a platform where strangers deploy code, that's the setup I'd check mine against.

## References

1. Josh Manders. [The post that started it](https://x.com/joshmanders/status/2117458724407415295), 3 November 2026, 03:27 UTC.
2. Josh Manders. [What the agents ran](https://x.com/joshmanders/status/2117530391183507484), 08:12 UTC.
3. Jake Runzer. [Confirmation and halting the harness](https://x.com/jakerunzer/status/2117474337682780643), 04:29 UTC.
4. Jake Cooper. [The apology and the ban on automated external testing](https://x.com/JustJake/status/2117486226718752878), 05:17 UTC.
5. Runtime Wire. [Railway cofounder apologizes after employee accused of trying to breach Primcloud](https://runtimewire.com/article/railway-cofounder-apologizes-after-employee-accused-of-trying-to-breach-primclou).
