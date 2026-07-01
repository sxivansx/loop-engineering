---
description: Catch me up on what the loop has done since I was last here, then resume work on the next open items
argument-hint: [optional focus, e.g. "just the summary" or "skip to work"]
---

# Sync Loop

The user is back on this project. Do two things, in order: **catch them up**, then **get the loop moving again**. If `$ARGUMENTS` says summary-only, stop after part 1; if it says skip-to-work, keep part 1 to two sentences.

## 1. Catch up (report, don't touch anything yet)

1. Read `LOOP.md` in full — goal, stop condition, bounds, and the `## Open`, `## Done`, `## Blocked` sections.
2. Establish what changed since the user was last here: `git log --oneline` since the last sync (record each sync as a line at the bottom of `LOOP.md` under `## Sync log`, and read the previous entry to know where to diff from; if there is no sync log yet, use the last ~15 commits), plus any items that moved between sections.
3. Give the user a short, plain-language status:
   - What got **done**, with the evidence the checker recorded.
   - What is **blocked** and why — these need the user's input; ask the specific questions.
   - What is still **open**, in the order the loop would take it.
   - Whether the **stop condition** is met, and how the loop is tracking against its bounds.
4. Append a `## Sync log` entry to `LOOP.md`: date, current HEAD commit, one-line status.

## 2. Resume (only after the summary is delivered)

5. If anything in `## Blocked` was just unblocked by the user's answers, move it back to `## Open` with the new context.
6. If `## Open` is empty, run this loop's discovery step to refill it before working.
7. Then run the normal cycle on the next item, following the Rules in `LOOP.md`:
   - **loop-maker** does exactly one item from `## Open`.
   - **loop-checker** independently verifies it with the real command and is the only one who moves it to `## Done`.
   - Run **loop-monitor** if it's due per the cadence in `LOOP.md`.
8. Keep cycling items while the user is present, updating `LOOP.md` after every cycle. Stop when the stop condition is met, a bound is hit, everything left is blocked on the user, or the user says stop.

Never mark work done yourself, never skip the checker, and never act on blocked items without the user's answer — same rules as every other cycle.
