# Loop Engineering

**Set up an autonomous, self-running AI coding agent in your project with one command.**

[![License: MIT](https://img.shields.io/badge/License-MIT-black.svg)](LICENSE)
[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-black.svg)](https://code.claude.com/docs/en/skills)
[![Agent Skills standard](https://img.shields.io/badge/Agent%20Skills-standard-black.svg)](https://agentskills.io)

Loop engineering is the practice of designing the system that prompts an AI agent for you, instead of prompting it yourself, turn by turn. You define a goal and a stop condition once. The loop finds the work, does it, verifies it with a separate agent, records what happened, and runs again until the goal is met.

This repo is an installable [Claude Code](https://claude.com/claude-code) skill. Run `/loop-engineering` inside any existing repo and it scaffolds the whole loop for you: it reads how the project already tests, builds, and tracks work, then drops in a state file, a maker agent, an independent checker, and a trigger, with the guardrails built in.

## Install

```bash
git clone https://github.com/sxivansx/loop-engineering ~/.claude/skills/loop-engineering
```

That makes `/loop-engineering` available in every project. For one project only, clone into that repo's `.claude/skills/` instead. The command name comes from the folder name, so keep it `loop-engineering`.

## Use

```text
/loop-engineering keep the main branch's CI green
```

Claude pins the goal and a verifiable stop condition, writes the state file, creates the maker and checker agents, picks a trigger, and runs one cycle by hand before anything is enabled. Nothing with side effects (scheduled jobs, pushes, deploys) happens without your confirmation.

It sets up:

```
your-project/
├── LOOP.md                 # goal, stop condition, bounds, rules, Open/Done/Blocked
├── .claude/agents/
│   ├── loop-maker.md       # does one unit of work
│   ├── loop-checker.md     # verifies it, independently
│   └── loop-monitor.md     # watches the whole loop for drift + runaway
└── trigger: /goal, /loop, a scheduled task, or a GitHub Action
```

Templates for each file and a complete [worked example](examples/keep-ci-green.md) ship with the skill.

## The six primitives

Every loop is the same six parts. Learn the parts, not one tool's syntax.

| Primitive | Role | Mechanism in Claude Code |
|---|---|---|
| **Trigger** | starts each cycle | `/goal` (work until a condition is met), `/loop`, a scheduled task, a hook, a GitHub Action |
| **State** | memory between cycles, on disk | a `LOOP.md` file, or an issue tracker via MCP |
| **Discovery** | finds what to work on | a triage step reading CI, issues, diffs, a queue |
| **Maker** | does one unit of work | a subagent that implements a single item |
| **Checker** | verifies the maker independently | a *different* subagent that runs the real check |
| **Stop condition** | ends the loop | a verifiable command, not a vibe |

For any loop that runs unattended, add a **monitor**: an oversight agent that watches the whole loop and halts it on drift (going off-goal), spinning (no net progress), or a crossed bound (too many cycles, too long). The maker and checker keep each item correct; the monitor keeps the loop pointed at the goal so it cannot quietly wander off and run for hours.

Optional add-ons: **worktrees** isolate parallel makers so they never edit the same file, and **MCP connectors** let the loop open PRs, update tickets, or post status. Add them only when the goal needs parallelism or external side effects.

## Guardrails

An autonomous loop is also an autonomous way to ship mistakes. These are built into the skill and not optional.

- **The checker is never the maker.** The agent that wrote the code does not get to declare it correct.
- **The stop condition is verifiable.** "Looks done" is not a stop condition. A command that exits 0 is.
- **State is written every cycle.** The model forgets between runs; the file does not.
- **A human gate stays on anything irreversible.** Merging, deploying, deleting, spending money. The loop drafts, a person confirms.
- **Blocked beats guessing.** When the loop cannot verify something, it flags it for a human instead of shipping a guess.

Two people can build the same loop and get opposite results: one moves faster on work they understand, the other avoids understanding the work at all. The loop does not know the difference. You do.

## Loop vs prompt vs context engineering

| | What you design | The unit | The question |
|---|---|---|---|
| **Prompt engineering** | a single instruction | one response | "What do I ask?" |
| **Context engineering** | what the agent knows | one run | "What goes in the context window?" |
| **Loop engineering** | the system around the agent | many runs over time | "How does it run, verify, and repeat without me?" |

A loop uses good prompts and good context inside it. Loop engineering wraps them into something that runs on its own.

## FAQ

**How is it different from prompt engineering?** Prompt engineering optimizes one instruction for one response. Loop engineering designs the system that issues many prompts over time and decides what to do next. Prompting is a part of a loop.

**What is the maker and checker split?** The maker does the work; a separate checker verifies it against the stop condition and the tests. Splitting them stops the loop from grading its own homework, the most common way autonomous agents ship broken work.

**What stops the loop from going off the rails?** A monitor agent runs alongside the maker and checker and watches the whole loop, not just each item. If the work drifts off the goal, stops making progress, or crosses a bound you set (max cycles or time), the monitor halts the loop and escalates to a human instead of letting it run for hours.

**Does it run unattended?** It can, but a human gate stays on anything irreversible (merging, deploying, deleting, spending money). The loop prepares and verifies the change; a person approves what can't be undone.

**What does it work with?** This skill targets Claude Code (`/goal`, `/loop`, scheduled tasks, subagents, hooks, MCP, GitHub Actions). The six primitives apply on any platform that can schedule a run and isolate work.

## Sources

- Peter Steinberger, *Beyond Vibe Coding*
- Boris Cherny (Claude Code, Anthropic)
- Addy Osmani, *Loop Engineering*: https://addyosmani.com/blog/loop-engineering/
- Claude Code skills: https://code.claude.com/docs/en/skills

## Author

**Shivansh Pandey**, frontend developer and designer building AI-assisted software. [X](https://x.com/shivansh_life) · [GitHub](https://github.com/sxivansx) · [LinkedIn](https://www.linkedin.com/in/shivansh-life)

## License

[MIT](LICENSE) © 2026 Shivansh Pandey
