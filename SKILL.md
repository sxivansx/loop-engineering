---
name: loop-engineering
description: Use when the user wants to "set up a loop", build a self-running or run-until-done agent, automate a recurring engineering task (triage, keep-CI-green, dependency bumps, backlog burndown), or stop hand-prompting an agent turn by turn. Sets up an autonomous engineering loop in the current project: a self-running system that discovers work, runs a maker agent and a separate checker agent, records state on disk, and re-triggers until a verifiable stop condition is met.
argument-hint: [what the loop should do]
---

# Loop Engineering

Set up a loop: an autonomous system that prompts the agent so you don't have to. You define the goal and the stop condition once; the loop finds work, does it, checks it, writes down what happened, and runs again until the goal is met. This skill stands that loop up in the current project.

A loop is built from six primitives and nothing else. Set up only the ones the goal needs.

| Primitive | Role in the loop | Claude Code mechanism |
|---|---|---|
| **Trigger** | starts each cycle, on a schedule or an event | `/loop`, scheduled tasks, hooks, GitHub Actions |
| **State** | the loop's memory between cycles, on disk | a markdown file (`LOOP.md`) or an issue tracker via MCP |
| **Discovery** | finds what to work on this cycle | a triage step that reads CI, issues, diffs, a queue |
| **Maker** | does one unit of the work | a subagent in `.claude/agents/` |
| **Checker** | verifies the maker, independently | a *different* subagent in `.claude/agents/` |
| **Stop condition** | the rule that ends the loop | a verifiable check the checker runs |

Two optional add-ons: **worktrees** (`git worktree`) isolate parallel makers so they don't edit the same files, and **MCP connectors** let the loop touch real tools (open a PR, update a ticket, post status). Add them only when the goal needs parallelism or external side effects.

## Setup procedure

Follow these steps when invoked. Confirm with the user before anything with side effects: scheduled jobs, pushes, ticket writes, deploys.

1. **Pin the goal and the stop condition.** Ask the user for the loop's one goal and the *verifiable* condition that means done (tests exit 0, the queue is empty, zero failing checks). A loop with no stop condition is the one thing you must never ship. If the user can't name one, set a bounded fallback (stop after N cycles) and say so out loud.

2. **Write the state file.** Copy `${CLAUDE_SKILL_DIR}/templates/LOOP.md` to the repo root and fill in goal, stop condition, cadence, and the human gate. It has three living sections the loop maintains: `## Open` (found, not done), `## Done` (with the evidence that verified it), `## Blocked` (needs a human). The loop reads this first every cycle and writes it last.

3. **Create the maker subagent.** Copy `${CLAUDE_SKILL_DIR}/templates/loop-maker.md` to `.claude/agents/loop-maker.md`. It does exactly one item from `## Open`, following the project's existing skills and conventions, and never marks its own work done.

4. **Create the checker subagent.** Copy `${CLAUDE_SKILL_DIR}/templates/loop-checker.md` to `.claude/agents/loop-checker.md`. It is a different agent from the maker, it runs the actual verification the stop condition names, and it is the only agent allowed to move an item to `## Done`.

5. **Define discovery.** Write the triage step that fills `## Open` each cycle. Specify what it reads (CI status, open issues, recent commits, a work queue) and how it turns findings into discrete, individually checkable items. Make it deterministic wherever you can.

6. **Choose isolation (optional).** If more than one maker runs per cycle, give each its own `git worktree` and branch so they never write the same file. One worktree per item.

7. **Wire connectors (optional).** If the loop must open PRs, update tickets, or post status, connect the matching MCP server. Otherwise the loop's output stays local and a human relays it.

8. **Set the trigger.** Pick the lightest mechanism that fits the cadence:
   - on demand, run-until-done in this session → `/loop`
   - periodic and unattended → a scheduled task (or cron / a GitHub Action for CI-side loops)
   - react to an event (push, PR, file change) → a hook or a GitHub Action

   Confirm with the user before creating any scheduled job.

9. **Dry-run, verify, then enable.** Run one cycle by hand. Read everything it produced. Confirm the checker actually blocks bad output and that state was written. Only then turn on the trigger.

## Guardrails (non-negotiable)

- **The checker is never the maker.** The agent that wrote the code does not get to declare it correct.
- **The stop condition is verifiable, not vibes.** "Looks done" is not a stop condition. A command that exits 0 is.
- **State is written every cycle.** If the loop forgets what it tried, it repeats itself. The model forgets; the file does not.
- **A human gate stays on anything irreversible.** Shipping, deploying, deleting, spending money. The loop drafts, a person confirms. An unattended loop makes unattended mistakes.
- **Blocked beats guessing.** When the loop can't verify or can't proceed, it writes to `## Blocked` and stops that item. It does not ship a guess.

## What the loop looks like once it's set up

```
your-project/
├── LOOP.md                   # goal, stop condition, cadence, Open/Done/Blocked
├── .claude/
│   └── agents/
│       ├── loop-maker.md      # does one unit of work
│       └── loop-checker.md    # verifies it, independently
└── (trigger: /loop, a scheduled task, or a GitHub Action)
```

## Anti-patterns: do not ship these

- A loop with no stop condition (runs forever, burns tokens).
- One agent that writes and grades its own work.
- A loop with no state file (re-derives everything, repeats work, forgets what failed).
- Unattended shipping with no human gate on irreversible actions.
- One loop carrying many unrelated jobs. One loop, one goal, one stop condition. Split the rest into their own loops.

## Files in this skill

- `templates/LOOP.md`: the state file to copy into the target repo.
- `templates/loop-maker.md`: the maker subagent definition.
- `templates/loop-checker.md`: the checker subagent definition.
- `examples/keep-ci-green.md`: a complete worked loop, from goal to trigger.
