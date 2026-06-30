# Loop Engineering

**Set up an autonomous, self-running AI coding agent in your project with one command.**

[![License: MIT](https://img.shields.io/badge/License-MIT-black.svg)](LICENSE)
[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-black.svg)](https://code.claude.com/docs/en/skills)
[![Agent Skills standard](https://img.shields.io/badge/Agent%20Skills-standard-black.svg)](https://agentskills.io)

Loop engineering is the practice of designing the system that prompts an AI agent for you, instead of prompting the agent yourself, turn by turn. You define a goal and a stop condition once. The loop then finds the work, does it, verifies it with a separate agent, records what happened, and runs again until the goal is met.

This repository is an installable [Claude Code](https://claude.com/claude-code) skill. Run `/loop-engineering` and it scaffolds a working loop in the current project: a state file, a maker agent, an independent checker agent, and a trigger, with the guardrails built in.

---

## Table of contents

- [Quick start](#quick-start)
- [What it sets up](#what-it-sets-up)
- [What is loop engineering?](#what-is-loop-engineering)
- [The six primitives of a loop](#the-six-primitives-of-a-loop)
- [Loop engineering vs prompt engineering vs context engineering](#loop-engineering-vs-prompt-engineering-vs-context-engineering)
- [Guardrails: how to stay the engineer](#guardrails-how-to-stay-the-engineer)
- [Worked example](#worked-example)
- [FAQ](#faq)
- [Sources](#sources)
- [Author](#author)
- [License](#license)

## Quick start

Install the skill at the personal level (available in every project) or the project level (committed to one repo).

```bash
# Personal: available across all your projects
git clone https://github.com/sxivansx/loop-engineering ~/.claude/skills/loop-engineering

# Or project-scoped: committed alongside one repo
git clone https://github.com/sxivansx/loop-engineering .claude/skills/loop-engineering
```

The skill's command name comes from its folder name, so keep the directory `loop-engineering` (the commands above already do). Then, in Claude Code, invoke it with the goal of the loop:

```text
/loop-engineering keep the main branch's CI green
```

Claude walks the setup: it pins the goal and a verifiable stop condition, writes the state file, creates the maker and checker agents, picks a trigger, and runs one cycle by hand before anything is enabled. Nothing with side effects (scheduled jobs, pushes, deploys) happens without your confirmation.

## What it sets up

```
your-project/
├── LOOP.md                   # goal, stop condition, cadence, Open/Done/Blocked
├── .claude/
│   └── agents/
│       ├── loop-maker.md      # does one unit of work
│       └── loop-checker.md    # verifies it, independently
└── (trigger: /loop, a scheduled task, or a GitHub Action)
```

The skill ships the templates for each of these files and a complete [worked example](examples/keep-ci-green.md).

## What is loop engineering?

Loop engineering is one level of abstraction above prompting. A prompt gets you one good response. A loop gets you a system that keeps producing verified results without you in the chair for each step.

The shift is concrete. The old way: you write a prompt, read the reply, write the next prompt, and you are the thing holding the agent on task. The loop way: you design a system that discovers work, hands it to an agent, checks the result with a second agent, writes the outcome to disk, and decides what to do next, on its own, until a stop condition is true.

This is harder than prompt engineering, not easier. The difficulty moves from "what do I ask?" to "how do I architect a system that runs safely without me watching every step?" The payoff is that you design the loop once and it keeps working while you do something else.

## The six primitives of a loop

Every loop is built from the same six parts. Learn the parts, not any one tool's syntax.

| Primitive | Role in the loop | Example mechanism |
|---|---|---|
| **Trigger** | starts each cycle, on a schedule or an event | `/goal`, `/loop`, a scheduled task, a hook, a GitHub Action |
| **State** | the loop's memory between cycles, stored on disk | a `LOOP.md` file, or an issue tracker via MCP |
| **Discovery** | finds what to work on this cycle | a triage step reading CI, issues, diffs, a queue |
| **Maker** | does one unit of the work | a subagent that implements a single item |
| **Checker** | verifies the maker independently | a *different* subagent that runs the real check |
| **Stop condition** | the rule that ends the loop | a verifiable command, not a vibe |

Two optional add-ons: **worktrees** isolate parallel makers so they never edit the same file, and **MCP connectors** let the loop touch real tools such as pull requests, issue trackers, and chat. Add them only when the goal needs parallelism or external side effects.

## Loop engineering vs prompt engineering vs context engineering

These are three different jobs at three different altitudes. They stack; they do not compete.

| | What you design | The unit | The question it answers |
|---|---|---|---|
| **Prompt engineering** | a single instruction | one response | "What do I ask to get a good answer?" |
| **Context engineering** | what the agent knows | one run | "What should be in the agent's context window?" |
| **Loop engineering** | the system around the agent | many runs over time | "How do I make the agent run, verify, and repeat without me?" |

A good loop uses good prompts and good context inside it. Loop engineering is what wraps them into something that runs on its own.

## Guardrails: how to stay the engineer

An autonomous loop is also an autonomous way to ship mistakes. These rules are built into the skill and are not optional.

- **The checker is never the maker.** The agent that wrote the code does not get to declare it correct.
- **The stop condition is verifiable, not vibes.** "Looks done" is not a stop condition. A command that exits 0 is.
- **State is written every cycle.** If the loop forgets what it tried, it repeats itself. The model forgets; the file does not.
- **A human gate stays on anything irreversible.** Shipping, deploying, deleting, spending money. The loop drafts, a person confirms.
- **Blocked beats guessing.** When the loop cannot verify or cannot proceed, it stops that item and flags it for a human rather than shipping a guess.

Two people can build the same loop and get opposite results: one uses it to move faster on work they understand, the other uses it to avoid understanding the work at all. The loop does not know the difference. You do.

## Worked example

[`examples/keep-ci-green.md`](examples/keep-ci-green.md) walks a full loop end to end: watch the default branch for failing CI, drive each failure back to green with a maker and an independent checker, open a fix PR, and leave the merge to a human. It shows the state file, the discovery step, both agents, and the trigger.

## FAQ

### What is loop engineering?
Loop engineering is designing an autonomous system that prompts an AI agent for you. Instead of writing each prompt by hand, you define a goal and a verifiable stop condition once, and the loop discovers work, makes a change, verifies it with a separate agent, records the outcome, and repeats until the goal is met.

### How is loop engineering different from prompt engineering?
Prompt engineering optimizes a single instruction for a single response. Loop engineering designs the system that issues many prompts over time and decides what to do next. Prompting is a part of a loop; loop engineering is the architecture around it.

### Do I need to be a developer to use this skill?
You need a project where work can be checked automatically (tests, a build, a lint, a clear pass or fail). The skill handles the structure. The most important input it asks you for is the stop condition: the check that proves the work is actually done.

### What is the maker and checker split?
The maker is the agent that does the work. The checker is a separate agent that verifies it against the stop condition and the project's tests. Splitting them stops the loop from grading its own homework, which is the most common way autonomous agents ship broken work.

### Does the loop run completely unattended?
It can run unattended, but a human gate stays on anything irreversible: merging, deploying, deleting, or spending money. The loop prepares the change and verifies it; a person approves the part that cannot be undone.

### What tools does loop engineering work with?
The six primitives are tool-agnostic. This skill targets Claude Code (using `/goal` to run until a condition is met, plus `/loop`, scheduled tasks, subagents, hooks, MCP, and GitHub Actions), but the same structure (trigger, state, discovery, maker, checker, stop condition) applies in any agent platform that can schedule a run and isolate work.

### What happens when the loop gets stuck?
It writes the item to a `Blocked` section in the state file with the reason and stops working that item. It does not guess past a check it cannot pass. A human picks up what is blocked on the next pass.

## Sources

Loop engineering as a named practice was articulated by Peter Steinberger and Boris Cherny, and synthesized by Addy Osmani.

- Peter Steinberger, *Beyond Vibe Coding*
- Boris Cherny (Claude Code, Anthropic)
- Addy Osmani, *Loop Engineering*: https://addyosmani.com/blog/loop-engineering/
- Claude Code skills documentation: https://code.claude.com/docs/en/skills

## Author

**Shivansh Pandey** is a frontend developer and designer who builds and ships AI-assisted software.

- X: [@shivansh_life](https://x.com/shivansh_life)
- GitHub: [@sxivansx](https://github.com/sxivansx)
- LinkedIn: [shivansh-life](https://www.linkedin.com/in/shivansh-life)

## License

[MIT](LICENSE) © 2026 Shivansh Pandey
