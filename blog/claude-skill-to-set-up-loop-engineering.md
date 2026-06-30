# I Wrote a Claude Skill That Sets Up Loop Engineering in Your Repo

For months I used my coding agent the same way most people do: write a prompt, read the reply, write the next prompt. I was the loop. I was the thing deciding what came next and noticing when it drifted off course.

Loop engineering is the fix, and I turned it into a Claude Code skill you can run in one command.

**Repo: [github.com/sxivansx/loop-engineering](https://github.com/sxivansx/loop-engineering)**

## What loop engineering actually is

Loop engineering is designing the system that prompts the agent for you, instead of prompting it yourself, turn by turn. You define a goal and a stop condition once. The loop then finds the work, does it, verifies it with a separate agent, writes down what happened, and runs again until the goal is met.

It is built from six parts: a trigger, a state file, a discovery step, a maker, a checker, and a stop condition. That is the whole vocabulary. The skill assembles them for you so you are not wiring it together by hand every time.

## What the skill does

Install it, then run it with the goal of your loop:

```text
/loop-engineering keep the main branch's CI green
```

From there it walks the setup and scaffolds a working loop in your repo:

1. It pins your goal and asks for the one thing most people skip: a verifiable stop condition. Not "looks done." A command that exits 0.
2. It writes a `LOOP.md` state file with three living sections, `Open`, `Done`, and `Blocked`. This is the loop's memory between runs, because the model forgets and the file does not.
3. It creates two agents in `.claude/agents/`: a `loop-maker` that does one item at a time, and a `loop-checker` that independently verifies the work. The checker is the only one allowed to mark something done.
4. It picks the lightest trigger for your cadence: `/loop` for on-demand runs, a scheduled task for unattended ones, or a GitHub Action for CI-side loops.
5. It runs one cycle by hand so you can watch it work before anything is switched on.

What you end up with:

```text
your-project/
├── LOOP.md
├── .claude/
│   └── agents/
│       ├── loop-maker.md
│       └── loop-checker.md
└── (trigger: /loop, a scheduled task, or a GitHub Action)
```

Nothing with side effects, like a scheduled job, a push, or a deploy, happens without your confirmation.

## How to install it

```bash
git clone https://github.com/sxivansx/loop-engineering ~/.claude/skills/loop-engineering
```

That makes `/loop-engineering` available in every project. If you would rather commit it to a single repo, clone it into that repo's `.claude/skills/` instead. The folder name becomes the command, so keep it `loop-engineering`.

## The guardrails I baked in

An autonomous loop is also an autonomous way to ship mistakes. So the skill refuses to set one up without these:

- The checker is never the maker. The agent that wrote the code does not get to say it is correct.
- The stop condition is a command that passes, not a vibe.
- State is written every cycle, so the loop never repeats work it already tried.
- A human gate stays on anything irreversible: merging, deploying, deleting, spending money.
- When the loop cannot verify something, it flags it for a human instead of guessing.

Here is the line I kept coming back to while building it. Two people can set up the exact same loop and get opposite results. One uses it to move faster on work they understand. The other uses it to avoid understanding the work at all. The loop does not know the difference. You do.

## Why I built it

I wanted to stop being the bottleneck for every step without handing over the judgment that makes the code mine. A loop does that when it is built right. It runs the boring cycles, and I stay the engineer who reads what it shipped and owns it.

If you have been hand-prompting your agent through the same task over and over, this is the thing I wish I had a month ago. Clone it, point it at one annoying recurring job, and keep the guardrails on.

**Get it: [github.com/sxivansx/loop-engineering](https://github.com/sxivansx/loop-engineering)**

---

*By [Shivansh Pandey](https://x.com/shivansh_life). The idea of loop engineering was articulated by Peter Steinberger and Boris Cherny, and synthesized by [Addy Osmani](https://addyosmani.com/blog/loop-engineering/).*
