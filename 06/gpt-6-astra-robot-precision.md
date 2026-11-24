---
title: "GPT-6 Astra can move a block almost perfectly. Precision is another matter"
url: "https://bosnadev.com/2026/09/06/gpt-6-astra-robot-precision"
author: "Mirza Pašić"
date: "2026-09-06"
topic: "Agents"
tags: ["openai", "robotics", "evaluation", "benchmarks"]
summary: "GPT-6 Astra completed 19 of 20 block-placement trials but only 2 of 20 precision insertions. Robot performance fell sharply as tolerances narrowed."
---

# GPT-6 Astra can move a block almost perfectly. Precision is another matter

GPT-6 Astra completed **19 of 20 trials** in which a pair of robot arms had to pick up a red block and place it inside a bowl. When asked to insert a puzzle piece into a matching groove, it succeeded **2 times out of 20**.[^1]

The two results show how much a smaller margin changes performance. Both tasks require the model to see, plan and control the robot. The puzzle leaves much less room for error. Astra often moved the piece to the right place, then missed the groove by a few millimetres.

[Robocurve](https://openai.robocurve.org/gpt-6-astra/) tested GPT-6 Astra, Claude Fable 5.1 and Claude Fable 5 on YAM robot arms under the same agent policy. Each model received images from three cameras and the robot's proprioceptive state, then controlled the arms through end-effector movements. The test gave each model 20 attempts at both tasks.[^1]

| Model       | Block into bowl | Puzzle into groove | Avg. time, bowl | Avg. time, puzzle |
| ----------- | --------------: | -----------------: | --------------: | ----------------: |
| GPT-6 Astra |       **19/20** |           **2/20** |         2.5 min |           3.4 min |
| Fable 5.1   |            8/20 |               2/20 |         6.8 min |           5.9 min |
| Fable 5     |            1/20 |               0/20 |         8.2 min |           7.9 min |

On this setup, Astra was reliable at coarse manipulation. Its performance fell sharply when placement had to be precise.

## A few millimetres change the task

Putting a block into a bowl allows several kinds of imprecision. The robot has to find the block, grasp it, move it over a large target and release it somewhere inside.

The round puzzle piece has to be grasped by its knob, moved to the correct part of the board and aligned with a circular groove. Robocurve's stage-level results show that Astra often positioned the piece above the deposit point before failing during insertion.[^1]

The same problem appears in computer-use agents. Reaching the correct application can be easy while clicking or dragging a small control remains unreliable. Software usually allows another attempt. A physical near miss can damage a component or knock an object off a table.

## The result is easy to inspect

Robot trials have an external outcome. The block ends up in the bowl, or the puzzle piece sits in its groove. A plausible-looking plan earns no credit when the object is in the wrong place.

Robocurve lists several limitations. Some bowl trials used different physical rigs, the human graders knew which model they were scoring, and the runs were not fully interleaved. Those differences make close comparisons between models less certain.[^1]

The large gap between Astra's two tasks is still useful. It shows that the same model and agent policy can handle a forgiving target in 19 of 20 trials and a precise insertion in only 2 of 20.

This kind of check also works outside robotics. Code can be executed, a circuit can be simulated, and a web task can be checked against the resulting application state. In each case, something outside the model decides whether the task succeeded.

## Astra was faster and cheaper in these trials

Astra's bowl attempts took an average of 2.5 minutes, compared with 6.8 minutes for Fable 5.1. Robocurve estimated the model cost at about $0.94 per Astra run and $2.12 per Fable 5.1 run.

On the puzzle task, Astra averaged 3.4 minutes and an estimated $1.36 per run. Fable 5.1 averaged 5.9 minutes and $2.18.[^1]

These are estimates based on list prices rather than invoices. Provider caching and billing can change the actual cost. Within those limits, the trials show why token price alone is a poor comparison for agents: a more expensive token can produce a cheaper task when the model needs fewer steps.

## The robot enforces its own limits

Robocurve runs the robot at 25% speed and relies on built-in safety features. The controller enforces those limits regardless of what the model tries to do.

Software agents need the same separation. The environment should enforce permissions instead of relying on a prompt to remind the model. A robot controller must also reject unsafe force or speed, even when the model expects a larger movement to finish the task.

## Precision is still the boundary

The block test shows that a general-purpose model can combine vision, planning and physical control well enough to repeat a forgiving manipulation task. The puzzle test finds the current limit.

Many physical jobs depend on repeatable precision: inserting components, connecting cables, handling lab equipment, assembling electronics, using tools and operating machines. A robot that gets close cannot do those jobs reliably.

In Robocurve's setup, Astra placed the block in the bowl 19 times and inserted the puzzle piece twice. The final alignment has to improve before the same approach can be dependable for work with tight tolerances.

### References

[^1]: [Robocurve, "GPT-6 Astra on robotic manipulation," September 2026.](https://openai.robocurve.org/gpt-6-astra/) Robocurve reports 120 trials across GPT-6 Astra, Claude Fable 5.1 and Claude Fable 5 on two bimanual manipulation tasks. It publishes completion and stage-level results, timing, output-token counts, estimated costs, transcripts, videos, rerun artifacts and methodological limitations.
