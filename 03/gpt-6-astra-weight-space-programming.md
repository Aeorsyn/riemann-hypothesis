---
title: "Weight-space programming: what GPT-6 Astra did to Qwen's weights"
url: "https://bosnadev.com/2026/11/03/gpt-6-astra-weight-space-programming"
author: "Mirza Pašić"
date: "2026-11-03"
topic: "Agents"
tags: ["openai", "evaluation", "interpretability"]
summary: "GPT-6 Astra edited 131,072 weights in Qwen3.5-0.8B and MATH-500 went from 7.9% to 48.1%. I went through the released logs to see what the edit does."
---

# Weight-space programming: what GPT-6 Astra did to Qwen's weights

On 2 November four researchers at MIT CSAIL, Rikiya Takehi, Jayden Teoh, Alex Zhang and Omar Khattab, published _Sparks of Weight-Space Programming_. They gave GPT-6 Astra the weights of a small open model, Qwen3.5-0.8B, and asked it to make the model better at maths. Astra changed 131,072 weights, with no gradient descent and no training run, and the model's MATH-500 score went from 7.9% to 48.1%.[^1]

The authors released everything: every turn of every run, the exact diffs and their hashes.[^2] I spent a day in those files. The edit is real. Hugging Face rebuilt it from the published diff, checked the hashes and scored 51.0% on MATH-500.[^3] The numbers in the post hold up too. What the edit does is narrower than "an AI rewrote another AI's weights":

- It stops the model getting stuck in a thinking loop. The model knows no more maths than before, and the authors say so further down their post.
- It is one direction per layer, found from activations and removed with a rank-one change. Arditi et al. published that recipe in 2024.[^4]
- Astra read the small networks with the same evidence a person can compute with one line of numpy.

The part I can't explain away is the networks Astra wrote from scratch, in one reply, with no tools. A Breakout policy written that way outscores PPO.

## Reading weights: small networks yes, large ones no

In Experiment 1 Astra gets the 17,253 weights of a small game-playing policy as text and no list of games to pick from. It names all four: Breakout, Pong, Pac-Man and Freeway, which it calls Frogger. It has no tools and spends about 800 to 1,000 reasoning tokens per run,[^5] which is far too few to simulate the network.

The prompt does tell Astra what each input is. It names the five input channels (agent, moving objects, reward targets, obstacles, previous frame) and gives the formula that maps a grid cell to an input index. With that layout, this one line computes what Astra described:

```python
np.linalg.norm(W1, axis=1).reshape(10, 10, 5)[:, :, 0]  # agent channel, norm per cell
```

![Four 10 by 10 grids of first-layer weight norms. Breakout lights up one row near the bottom, Pong one column, Pac-Man a maze, Freeway a vertical band.](https://media.bosnadev.com/hub/f8ac667658a1ba43/wsp-first-layer-1520x500.webp "Computed from the authors' released networks, exp1_which_game, commit 4d09ba6.")

The norm is large only on cells the agent can reach. Weights for cells the agent never visits stay near their starting size, so training draws the play area into the first layer. Sean Goedecke made the same argument the next day.[^6] adiant211 tested it on the same networks with a fixed three-rule readout and no LLM. That got 3 of 4 and missed Freeway, and the tool was told the four genres.[^7] Astra wasn't, so its open answer is the harder test.

| Experiment                    | What Astra had                           | Astra's result                                 | Check without an LLM                                        |
| ----------------------------- | ---------------------------------------- | ---------------------------------------------- | ----------------------------------------------------------- |
| 1. Which game                 | All weights, named channels, no tools    | 4 of 4, open answer                            | 3 of 4 from a norm map and three rules, told the genres[^7] |
| 2. Which shape (DDPM)         | All weights, no forward pass             | 3 of 3, drawn as outlines                      | 3 of 3 from Sobel edge profiles of the output layer[^7]     |
| 3. Language and copying heads | Middle 4 blocks, Python, no forward pass | Language right 6 of 6, all copying heads found | 2 of 3 networks exact with signed QK-OV composition[^7]     |
| 7. Which model is sycophantic | Weights of both Gemma copies             | Chance in all four attempts                    | None                                                        |
| 7, relaxed                    | Attention patterns on texts it wrote     | 3 of 3 pairs                                   | None                                                        |

Experiment 3 is the best read in the set. The input and output layers are removed, so no weight tells you which character is which. Astra multiplies the attention matrices together and finds the induction heads, the method from Elhage et al.[^8] The post says the team never mentioned that method, and I checked the prompt: it doesn't name QK, OV or the paper. It does define the target in its answer labels, `'copy' (induction: attends to the character right after an earlier occurrence...)`, and it gives the three possible languages as a closed list.[^9] Astra's stated confidence ran from 0.48 to 0.62.

Experiment 7 is the one to read before any claim about safety audits. One Gemma copy was trained to agree with 100% of wrong guesses, the other agrees with 1 to 3%. From weights alone, Astra couldn't tell them apart in four attempts. It got all three pairs right only once it could see attention patterns on texts it wrote.[^10] The post lists "auditing without evaluation" as a future use, and its own hardest audit needed a look at the model running.

## The Qwen edit

The run that produced the 48.1% took 158 turns, 80 Python runs and 20.3 hours.[^11] The final edit changes two tensors, `layers.15.mlp.down_proj.weight` and `layers.19.mlp.down_proj.weight`, each 1,024 × 3,584. Each column is what one MLP neuron writes into the residual stream. Astra's final report gives the formula:[^12]

```text
W' = W − 6 · u · (m ⊙ uᵀW)
```

`u` is a unit vector, the direction of repetition at that layer's output. Astra found it by comparing four pairs of contexts, one where the model was making progress and one where it was repeating itself. `m` is a mask that keeps 64 columns, the neurons that write most along `u`.

Take one of those columns, `w`. Its component along `u` was `uᵀw`, and after the edit it is `−5 uᵀw`. "Removed the direction" undersells it. Those neurons used to push the model toward repetition, and now they push away from it, five times as hard. adiant211 found the same factor.[^7]

The textbook version is `W' = W − uuᵀW` on every column, which sets the component to zero.[^4] Astra worked its way there: it ablated a single neuron first, then removed the direction from one layer, then from both. At full removal on both layers it measured a 24.2-point gain (SE 4.0) on 128 practice problems. Then it raised the dose and narrowed the edit to 64 columns.

The size of this edit depends on what you count:

| Measure                           | Value                                                             |
| --------------------------------- | ----------------------------------------------------------------- |
| Share of all weights touched      | 0.016% (0.017% of the 752M text weights[^7])                      |
| Change to the two edited matrices | 10.6% and 8.1%                                                    |
| Values Astra chose                | Two 1,024-dimensional directions, 128 column indices and one dose |

The last row is my count, not the post's. The 131,072 new values are not independent. Once you fix `u`, `m` and the dose, `W` gives you all of them, and each change is rank one: adiant211 measured σ₁/σ₂ at about 1,500 to 1,800. "Astra rewrote 131,072 weights" makes it sound like 131,072 decisions. Astra picked one direction per layer and a strength.

## The prompt pointed at the loop

Lucas Beyer read the prompt and noticed this line:[^13]

> if the model often fails because it repeats itself and never finishes, finding and fixing what drives that repetition counts

That is from the prompt of the 50-pass run.[^14] Coauthor Jayden Teoh replied that Claude Code added the hint while expanding a short assignment, without further human feedback.[^13] The prompt also lays out the method: diagnose failures first, localise them causally, test the dose, and compare paired runs on problems the edit wasn't tuned on. The forward-pass tool reports `cut_at_max_tokens` on every generation pass, and in pass 1 Astra saw 85 of its 96 answers cut off. It didn't have to dig to find the loop.

The logs also hold runs with prompt v4, which drops the hint. These are the rows from the results table:[^15]

| Run               | Prompt       | Passes | MATH-500 | GSM8K |
| ----------------- | ------------ | ------ | -------- | ----- |
| No edit submitted | v3           | 5      | 7.9      | 13.8  |
| p10c              | v3 with hint | 10     | 12.3     | 25.7  |
| p25c              | v3 with hint | 25     | 14.1     | 25.1  |
| p50c              | v3 with hint | 50     | 48.1     | 67.1  |
| No edit submitted | v4, no hint  | 10     | 7.9      | 13.8  |
| Not named         | v4, no hint  | 25     | 14.1     | 25.1  |

The v4 rows aren't in the post, and the release leaves those runs out. The 25-pass v4 scores match p25c to the decimal, so they may be the same edit, and I can't tell from what was released. There is no 50-pass v4 run. That's the one that would show whether Astra finds the loop fix without being told to look for it.

Each budget is also a single run, and the three v3 edits are different kinds of edit. The 10-pass run removes a direction from 30 matrices, the 25-pass run zeroes one attention head, and the 50-pass run is the edit above. IFEval goes 49.0, 39.9, 62.3 across them. I read that as three separate attempts, with no trend you could plot against budget.

## What the 48.1% measures

The unedited model hits the 32,768-token limit on 93.0% of MATH-500 answers. The edited model hits it on 33.1%. Most of the gain is the model finishing its answers. The authors say this in the Gemma section: editing "could remove an obstacle that hid an ability, but it could not add an ability the model lacked".[^1]

The released per-problem files let you check this one problem at a time. I compared the two models on each problem, averaged over samples:[^11]

| Benchmark | Problems better | Problems worse | Problems same | Paired SE of the gain |
| --------- | --------------- | -------------- | ------------- | --------------------- |
| MATH-500  | 306             | 0              | 194           | 1.7 points            |
| GSM8K     | 916             | 4              | 399           | 1.1 points            |

No MATH-500 problem got worse. The practice data agrees: on the 114 practice problems the base model already finished, the edit gained nothing, with 3 rescued and 3 lost. On the 1,038 it cut off, accuracy went from 1.4% to 56.7%.[^16]

That suggests a better question: when an answer now finishes, how often is it right? The post doesn't chart this, but its numbers give it. Divide the accuracy gain by the drop in cut-off answers:

![Bar chart of accuracy gain divided by the drop in cut-off answers, per benchmark. MATH-500 67%, GSM8K 66%, IFEval 57%, AMC23 47%, OlympiadBench 39%, Minerva 33%, GPQA Diamond 24%, MMLU-Pro 18%, AIME 2025 6%, AIME 2024 2%.](https://media.bosnadev.com/hub/701540310cc2bf2a/wsp-finished-answers-1520x740.webp "Computed from exp8_math/eval/unedited/merged.json and eval/p50c/merged.json, commit 4d09ba6. It assumes the gain comes from newly finished answers.")

This was the most useful number I got from the logs, because it shows how much ability the loop was hiding. On grade-school maths and MATH-500 it was hiding a lot. On AIME the edited model finishes a third of its answers and gets almost none right. On GPQA Diamond the newly finished answers score about what you'd get by guessing among four options. Part of the 3× on IFEval is mechanical: the grader checks the text after `</think>`, so an answer that never closes its thinking has nothing to grade.

The Gemma 3 270M control fits this reading. Gemma rarely loops, only 8.5% of its answers reach the limit, and Astra's edits moved its MATH-500 score from 6.8% to at most 7.7%.[^15] On a model without the loop, the same process found nothing to fix.

Qwen's own model card warns about this. It says Qwen3.5-0.8B "is more prone to entering thinking loops", and that this happens even with the recommended sampling settings.[^17] The evaluation used the card's top-k 20 and presence penalty 1.5, with temperature 0.6 instead of the card's 1.0. So the 7.9% baseline is a known weakness of this model, and Astra fixed it in the weights.

The fix is useful. It is permanent, it targets a real failure, and it was cheap. I added up the run's request files: the search used about 6,000 generations and 145M generated tokens. At the rate of the Hugging Face re-run, 43.4M tokens in 51 minutes on one H200,[^3] that is about three GPU-hours plus Astra's own tokens. It needed no training, but it did get feedback from all those generations.

## Writing weights from scratch

Experiment 5 surprised me. Astra gets the game rules and the source file and writes all 20,486 weights of a 600 → 32 → 32 → 6 tanh network in one reply, with no tools, feedback or training. For Breakout the reply is 20,567 tokens long and 815 of the weights are nonzero.[^18] adiant211 rebuilt the networks bit for bit and got the same scores.[^7]

The network reads like a compiled program. Four first-layer units compute where the ball will land for each direction it can travel, with the wall bounces written into the coefficients. The second layer decides between moving left and right.

The post's headline here is 517.4 against 15.8 for PPO. The evaluation file shows where that ratio comes from.[^19] All runs use sticky actions at 0.1, a 10,000-step cap and MinAtar 1.0.15:

| Breakout policy            | Mean score | Mean episode length | Score per step |
| -------------------------- | ---------- | ------------------- | -------------- |
| Astra, one reply           | 517.4      | 7,417               | 0.070          |
| PPO, 500M steps, best seed | 25.3       | 282                 | 0.090          |
| PPO, 100M steps, best seed | 15.8       | 178                 | 0.088          |

PPO scores faster per step and then drops the ball. Astra's policy rarely loses the ball: 517 of its 1,000 episodes hit the 10,000-step cap.[^7] With a longer cap the ratio would grow, so the 33× tells you about survival and the cap more than about how good the policy is. It is still a network that reacts to one frame, with no loop and no simulator, written blind, and it keeps the ball in play for thousands of steps.

The other games keep this in proportion. PPO at 500M steps beats Astra on Seaquest (88.9 against 53.1) and narrowly on Asterix (55.2 against 53.2). These are small MLPs on 10 × 10 grids. A task that compresses into a short program can be written down as weights, and Tracr showed that compile step in 2023 with a formal language as input.[^20] Here the input is English, and a general model does the compiling.

Experiment 6 does the same thing with tools. Astra builds a 1,480,274-weight Go network through a Python sandbox, layering in liberties, captures, atari and eyes. With no forward passes it wins 42.6%, 26.3% and 36.3% of games against three PPO networks of the same size. With 50 forward passes it wins 74.2%, 70.0% and 78.5%.[^21] Tree search isn't allowed, and a fully connected MLP is a weak shape for Go, so the opponents aren't strong. Experiment 4 is a one-shot edit to 6,073 of 171,664 weights that adds a back-up path to a Kinetix car policy. The authors report success going from 0% to 100%. adiant211 didn't re-run that one, and neither did I.

Chandan Singh ran a related test with Claude on tasks with known algorithms.[^22] Across 959 written programs, all ten algorithmic tasks reached 100% within three attempts. His report also records a failure. On language tasks the agent drifted from writing mechanisms to fitting them, and on SNLI it set its readout with logistic regression on the evaluation set. Any harness that gives an agent many rounds of feedback can drift the same way.

## What others found

The post reached nearly 2,900 points and over 850 comments on Hacker News.[^23]

| Who                  | Position                                                                                                                                                                                                                         | Source       |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ |
| Neel Nanda           | Astra is "surprisingly capable of doing old school mechanistic interpretability research rather than anything spooky"                                                                                                            | [^24]        |
| Lucas Beyer          | Read the prompt and pointed out the loop hint                                                                                                                                                                                    | [^13]        |
| Alex Zhang, coauthor | The team first thought Astra had edited a new maths skill into Qwen. They found it used a trick based on activation steering                                                                                                     | [^13], [^25] |
| Adam Karvonen        | Astra "used code to perform some standard interp techniques, found a looping direction, and ablated it from the weights"                                                                                                         | [^25]        |
| Sean Goedecke        | A smart human ML researcher could do all of these tasks                                                                                                                                                                          | [^6]         |
| Alex Krentsel        | Re-ran Experiment 5 across model versions and called it "a new, step-change emerging capability": 22× the baseline for Astra, 20× for Opus 5.5, 18× for GPT-6.1 Sol, lower for older models                                      | [^26], [^27] |
| adiant211            | Plain tools and a recipe recovered the loop direction on a CPU in 368 seconds: about 0.71 cosine with Astra's, 39 and 29 of the 64 neurons shared, about 75% of the loop suppression. A random edit of the same size did nothing | [^7]         |
| Hugging Face         | Rebuilt the edit and re-ran full MATH-500: 51.0%, with 31.6% of answers hitting the token limit                                                                                                                                  | [^3]         |
| r/LocalLLaMA         | Experiments 1 and 2 read structured first and last layers. A 7.9% baseline with 93% of answers cut off is a pathology, and the fix is Arditi-style ablation                                                                      | [^28]        |
| yorwba on HN         | "it succeeded using standard techniques also used by humans when they read and write neural network weights"                                                                                                                     | [^29]        |
| The Register         | Reported the claims, including the authors' recursive self-improvement framing, with no outside critic                                                                                                                           | [^30]        |

The adiant211 result doesn't make Astra's run trivial. That pipeline starts from Astra's final report, so it already knew the recipe, and its author says as much: a cheap model or a person picks the question and the recipe, and tools do the reading. Krentsel's results, for their part, show that writing weights in one reply got much better in recent models. Neither shows a new way of understanding networks. Both things are true: the capability is new at this level, and the methods are old.

## Where "natively" holds

The post's title says Astra reads and writes weights natively. The experiments split into three groups:

- **No tools, small networks.** Experiments 1, 2, 4 and 5. This is close to native. The networks either have structure that shows in the weights or do a task that compiles to a short program.
- **Python, no forward pass.** Experiment 3. Astra does linear algebra in a sandbox and decides what to compute.
- **Python plus forward passes.** Experiments 6, 7 and 8. This is an agent doing interpretability with activations, generations and paired tests. The result lands in the weights, but the information comes from running the model.

The Qwen result is in the third group. The post's closing ideas, recursive self-improvement and grafting circuits from one model into another, build on results like that one. The Gemma result and the failed weights-only audit in Experiment 7 cut against that step, and I wouldn't cite the post for those ideas yet.

These tests would settle more than another demo:

1. **Remove the hint at full budget.** Run prompt v4 at 50 passes with several seeds, and report every run, including the ones that submit nothing.
2. **Add a decoding baseline.** Score the base and edited Qwen side by side with a repetition penalty or a loop detector as a third arm. That shows what the edit adds beyond a sampling change.
3. **Break the labels.** Repeat Experiment 1 with the channel names removed and the channel order shuffled, so a norm map no longer tells you which channel is the agent.
4. **Blind the audit.** Plant a backdoor with a known trigger. Give Astra the weights only, then probes, and report false positives on clean models. That tests the auditing claim directly.
5. **Lift the cap.** Score Breakout without the 10,000-step limit and report survival and score rate separately.
6. **Use models with several failures.** Gemma suggests a single-direction fix only works when a single direction carries the failure.

The authors say they are building a benchmark and a harness. I hope it reports every run, fixes the prompts in advance and includes models with no easy bug. They released 1,125 audited turns with exact diffs and hashes,[^2] which is why I could check their work this closely. Most of what I've written here comes from their own files.

If you maintain small open models, you can use this now. A general agent with access to activations can find a rank-one fix for a known failure in about a day. Treat what it finds as a patch: keep the base checkpoint, hash the edit and run per-item regressions on the tasks you care about. And before you read a 6× gain as new skill, check how many answers were finishing before.

[^1]: Rikiya Takehi, Jayden Teoh, Alex Zhang, Omar Khattab. [Sparks of Weight-Space Programming](https://rikiyat.github.io/blog/weight-space-programming/). MIT CSAIL, 2 November 2026.

[^2]: [RikiyaT/lm_understand-logs](https://github.com/RikiyaT/lm_understand-logs). The complete logs. I worked from commit 4d09ba6.

[^3]: [RikiyaT/Qwen3.5-0.8B-weight-space-programming](https://huggingface.co/RikiyaT/Qwen3.5-0.8B-weight-space-programming). Model card, with the re-run by Ahsen Khaliq of Hugging Face.

[^4]: Andy Arditi et al. [Refusal in Language Models Is Mediated by a Single Direction](https://arxiv.org/abs/2406.11717). 2024.

[^5]: [Experiment 1 README](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp1_which_game/README.md).

[^6]: Sean Goedecke. [LLMs are not superhuman at reading and writing neural network weights](https://seangoedecke.com/llms-are-not-superhuman-at-reading-and-writing-neural-network-weights/). 3 November 2026.

[^7]: adiant211. [Weight-space programming, checked on a CPU](https://github.com/adiant211/wspacetest).

[^8]: Nelson Elhage et al. [A Mathematical Framework for Transformer Circuits](https://transformer-circuits.pub/2021/framework/index.html). 2021.

[^9]: [Experiment 3 prompt](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp3_language/runs/python_r0/turns/t001/prompt.txt).

[^10]: [Experiment 7 README](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp7_sycophancy/README.md).

[^11]: [Experiment 8 README](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp8_math/README.md).

[^12]: [Astra's final report, 50-pass run](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp8_math/runs/p50c/work_notes/final_report.md).

[^13]: Ksenia Se. [FOD#174: What is Weight-Space Programming and why it suddenly blew up the internet?](https://www.turingpost.com/p/what-is-weight-space-programming-and-why-it-suddenly-blew-up-the-internet) Turing Post, 2 November 2026.

[^14]: [Experiment 8 prompt, 50-pass run](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp8_math/runs/p50c/turns/t001/prompt.txt).

[^15]: [Experiment 8 results table, excerpt](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp8_math/eval/RESULTS_excerpt.md).

[^16]: [Run state for the 50-pass run](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp8_math/runs/p50c/state.json), output of Python run 80.

[^17]: Qwen. [Qwen3.5-0.8B model card](https://huggingface.co/Qwen/Qwen3.5-0.8B).

[^18]: [Experiment 5 README](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp5_write_from_scratch/README.md).

[^19]: [Experiment 5 evaluation data](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp5_write_from_scratch/eval/minatar_eval.json).

[^20]: David Lindner et al. [Tracr: Compiled Transformers as a Laboratory for Interpretability](https://arxiv.org/abs/2301.05062). 2023.

[^21]: [Experiment 6 README](https://github.com/RikiyaT/lm_understand-logs/blob/4d09ba6d6a7e891dd8063974dfe81b3d78e3494f/exp6_go_from_scratch/README.md).

[^22]: Chandan Singh. [Weight-Space Programming from scratch on simple tasks](https://csinva.io/weight-space-programming-autoresearch/analyze/toy_examples/report.html). 2 November 2026.

[^23]: [Sparks of Weight-Space Programming](https://news.ycombinator.com/item?id=50341174). Hacker News.

[^24]: [GPT-6 Astra Edits AI Weights Directly, Math Score Jumps 6x](https://shiftdelete.net/en/gpt-6-astra-weights). ShiftDelete, 3 November 2026. Quotes Neel Nanda.

[^25]: [Replication Confirms MIT Weight Editing Study as Authors Clarify Loop Fix](https://huggingnews.com/ai/replication-confirms-mit-weight-editing-study-as-authors-clarify-loop-fi-cd8a8f15). HuggingNews, 3 November 2026.

[^26]: Alex Krentsel. [Post on X](https://x.com/AlexKrentsel/status/2117376683724660898). 2 November 2026.

[^27]: Curtis Pyke. [Weight-Space Programming: GPT-6 Astra Edits Qwen](https://kingy.ai/blog/weight-space-programming-ai-agent-model-weights/). Kingy AI, 2 November 2026.

[^28]: swyx. [LLMs can read & write neural network weights: "Weight Space Programming" cuts Autoresearch out of the RSI loop](https://latent.space/p/ainews-llms-can-read-write-neural-network-weights-weight-space-programming-cuts-autoresearch-out-of/). Latent Space, 3 November 2026. Includes the Reddit recap.

[^29]: yorwba. [Comment on Hacker News](https://news.ycombinator.com/item?id=50343052).

[^30]: Thomas Claburn. [Forget retraining - AI can now tweak another AI's weights](https://www.theregister.com/ai-and-ml/2026/11/03/forget-retraining-ai-can-now-tweak-another-ais-weights/5307105). The Register, 3 November 2026.
