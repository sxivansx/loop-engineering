# Loop Engineering: How to Set Up a Self-Running AI Coding Agent

Most people still use AI coding agents the same way they used them two years ago: write a prompt, read the reply, write the next prompt. You are the loop. You are the thing holding the agent on task, deciding what comes next, and noticing when it went sideways.

Loop engineering is what happens when you stop doing that by hand.

**Loop engineering is the practice of designing the system that prompts the agent for you.** You define a goal and a stop condition once. The system discovers work, makes a change, checks it, writes down what happened, and runs again, until the goal is met. The job moves from writing a good prompt to designing a good loop.

This post explains the idea and then sets one up in your own project with a single command, using an open-source Claude Code skill.

> The skill is here: **[github.com/sxivansx/loop-engineering](https://github.com/sxivansx/loop-engineering)**

## Why this matters

A prompt gets you one good response. A loop gets you a system that keeps producing verified results without you in the chair for each step.

That is a real shift in where the difficulty lives. Prompt engineering asks "what do I type to get a good answer?" Loop engineering asks "how do I architect something that runs, verifies its own output, and repeats, safely, without me watching every step?" That second question is harder. It is also where the payoff is, because you answer it once and the loop keeps working while you do something else.

## The six primitives

Every loop is built from the same six parts. They are tool-agnostic. Learn the parts, not one tool's syntax.

1. **Trigger.** What starts each cycle: a schedule, an event, or a run-until-done command.
2. **State.** The loop's memory between cycles, stored on disk. The model forgets between runs; a file does not.
3. **Discovery.** How the loop finds what to work on this cycle: reading CI, open issues, recent commits, a queue.
4. **Maker.** The agent that does one unit of the work.
5. **Checker.** A separate agent that verifies the maker's output. Never the same agent that wrote it.
6. **Stop condition.** The verifiable rule that ends the loop. A command that exits 0, not a feeling that it looks done.

Two optional add-ons handle the harder cases: **worktrees** isolate parallel makers so they never edit the same file, and **connectors** (over MCP) let the loop open a pull request, update a ticket, or post status.

The single most important part is the stop condition. A loop without one either runs forever and burns tokens, or stops at the wrong moment and ships something half-done. If you cannot state how the loop knows it is finished, you do not have a loop yet.

## Setting one up in one command

The [loop-engineering skill](https://github.com/sxivansx/loop-engineering) scaffolds all of this for you in Claude Code. Install it once:

```bash
git clone https://github.com/sxivansx/loop-engineering ~/.claude/skills/loop-engineering
```

Then invoke it with the goal of your loop:

```text
/loop-engineering keep the main branch's CI green
```

From there, Claude walks the setup:

1. It pins your goal and asks for the verifiable stop condition.
2. It writes a `LOOP.md` state file with three living sections: `Open`, `Done`, and `Blocked`.
3. It creates two agents in `.claude/agents/`: a `loop-maker` that does one item at a time, and a `loop-checker` that independently verifies the work and is the only one allowed to mark something done.
4. It picks the lightest trigger that fits your cadence: `/loop` for on-demand runs, a scheduled task for unattended ones, or a GitHub Action for CI-side loops.
5. It runs one cycle by hand so you can see it work before anything is switched on.

Nothing with side effects (a scheduled job, a push, a deploy) happens without your confirmation. You can read the full structure it produces in the repo's [README](https://github.com/sxivansx/loop-engineering#what-it-sets-up), and a complete end-to-end loop in the [keep-CI-green example](https://github.com/sxivansx/loop-engineering/blob/main/examples/keep-ci-green.md).

## The part nobody should skip

An autonomous loop is also an autonomous way to ship mistakes. The skill builds in five guardrails, and they are worth stating plainly because they are the difference between a loop that helps and one that quietly breaks your codebase:

- The checker is never the maker. The agent that wrote the code does not get to say it is correct.
- The stop condition is a command that passes, not a vibe.
- State is written every cycle, so the loop never repeats work it already tried.
- A human gate stays on anything irreversible: merging, deploying, deleting, spending money.
- When the loop cannot verify something, it flags it for a human instead of guessing.

There is a sharper way to say the real risk. Two people can build the exact same loop and get opposite results. One uses it to move faster on work they understand. The other uses it to avoid understanding the work at all. The loop does not know the difference. You do.

## Build the loop, stay the engineer

The job is changing from "the person who prompts the agent" to "the person who designs the system that prompts the agent." That is more responsibility, not less. You still own the code the loop ships. You still have to read it. The loop just stops making you the bottleneck for every single step.

Set one up, keep the guardrails on, and stay the engineer.

**Get the skill: [github.com/sxivansx/loop-engineering](https://github.com/sxivansx/loop-engineering)**

---

*Written by [Shivansh Pandey](https://x.com/shivansh_life). The concept of loop engineering was articulated by Peter Steinberger and Boris Cherny, and synthesized by [Addy Osmani](https://addyosmani.com/blog/loop-engineering/).*
