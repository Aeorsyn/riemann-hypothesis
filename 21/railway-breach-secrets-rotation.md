---
title: "Railway was breached. Every customer has to rotate secrets, and deploys are paused"
url: "https://bosnadev.com/2026/10/21/railway-breach-secrets-rotation"
author: "Mirza Pašić"
date: "2026-10-21"
topic: "Server administration"
tags: ["security", "deployment", "sre", "railway"]
summary: "Railway told every customer to rotate all secrets while deploys are paused. What it has said so far, the first reports of stolen keys, and what I'd do tonight."
---

# Railway was breached. Every customer has to rotate secrets, and deploys are paused

Today [Railway](https://railway.com/) told every customer to roll all their environment variables, API keys and private registry credentials. The [same notice](https://blog.railway.com/p/security-disclosure-wednesday-october-21st) says all deployments are paused.

Those two lines don't fit together. A Railway service reads a new value when you deploy it, and right now you can't deploy. If you rotate a key, your running app keeps the old one until the pause ends, or breaks the moment the old key stops working. If you wait, a key that may be in someone else's hands keeps working.

## What Railway has said

Not much yet. An attacker got into Railway's systems and may have read customers' stored secrets. Railway hasn't said how they got in, how many projects they touched or which secrets they read, and it has promised more detail later.

For most of the day the [status page](https://status.railway.com/) described a platform outage with builds failing. It first called this a security incident at 21:59 UTC. The notice went out on X a few minutes earlier, at 21:51.

## Keys are already being used

Developers on X and in the [Hacker News thread](https://news.ycombinator.com/item?id=50188228) started reporting OpenAI and OpenRouter keys spent by someone else hours before Railway said anything. Others said Anthropic or OpenAI had revoked their keys.

At 21:11 UTC Nevo David shut down Postiz for all its customers. He wrote on X that Anthropic and Stripe had reported its keys as compromised. Postiz runs on Railway, and his post went up 40 minutes before Railway's.

So the AI providers and Stripe knew about leaked keys before Railway's customers did. I'd like to know when Railway told them, and why customers heard last.

## What I'd do tonight

If you run anything on Railway, I wouldn't wait for deploys to come back:

1. **Revoke at the issuer.** Changing a variable in Railway doesn't kill the old key. Revoke it at OpenAI, Anthropic, Stripe, AWS or wherever it came from, and accept that the service using it goes down until you can deploy.
2. **Start with keys that spend money.** AI provider keys, payment keys and cloud credentials first, then database passwords and webhook secrets.
3. **Put a spend cap on every AI key** you create to replace them, one key per service.
4. **Check usage since yesterday** on every provider dashboard, and look through your GitHub audit log and the apps that have access to your repositories.
5. **Tell your own users** if a service will be down, on a channel that doesn't run on Railway.

Then, when deploys return, put the new values in and deploy everything.

## After tonight

Your host holds your secrets and decrypts them on every deploy, so a breach there reaches your keys whatever you do in your own code. This could happen on any platform, and I'd plan for a day when the keys leak and deploys stop at the same time:

- Give every AI key a hard spend limit, and give each service its own key.
- Give database users the rights they need and no more, and keep databases off the public internet.
- Time a full rotation once and write down what breaks.
- Know where you'd run each service if your host froze, and which ones can stay down while you move.

Railway says it will publish a full report. The parts I'm waiting for are how the attacker got in and whether they could read source code as well as variables.

## References

1. Railway. [Security Disclosure: Wednesday October 21st](https://blog.railway.com/p/security-disclosure-wednesday-october-21st). The notice to rotate secrets, and the deploy pause.
2. Railway. [Status page](https://status.railway.com/). The outage entries and the security incident from 21:59 UTC.
3. Hacker News. [Security Disclosure](https://news.ycombinator.com/item?id=50188228). The discussion, with customers reporting misused keys.
