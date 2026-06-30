# Loop: <name>

**Goal:** <one sentence: what this loop exists to accomplish>
**Stop condition:** <a verifiable check that means done, e.g. `npm test` exits 0 and `## Open` is empty>
**Cadence:** <on-demand `/loop` | daily scheduled task | on push via a GitHub Action>
**Human gate:** <what a person must approve before it ships, e.g. PR merge, deploy>

_The loop reads this file at the start of every cycle and writes it at the end._

## Rules (every agent in this loop follows these)
- One item per cycle. Finish it or block it, never start a second.
- The maker never marks its own work done. Only the checker moves an item to `## Done`, and only after running the real check.
- Verify by running the command, not by reading a description. No green, no Done.
- When you cannot verify or cannot proceed, write it to `## Blocked` with the reason and stop. Do not guess past it.
- Nothing irreversible (merge, deploy, delete, spend) happens without the human gate above.

## Open
<!-- Work discovered this cycle, not yet done. One discrete, checkable item per line. -->

## Done
<!-- Moved here only by the checker, each with the evidence that verified it. -->

## Blocked
<!-- Needs a human. Say why. The loop does not guess past these. -->
