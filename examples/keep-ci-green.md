# Example loop: keep CI green

A complete loop, from goal to trigger. It watches for failing CI on the default branch and drives each failure back to green, with a human approving every merge.

## 1. Goal and stop condition

- **Goal:** keep the default branch's CI passing.
- **Stop condition:** the latest CI run on `main` is green and `## Open` is empty.
- **Cadence:** every push to `main`, plus a daily catch-up run.
- **Human gate:** a person merges the fix PR. The loop never merges.

## 2. State file (`LOOP.md`)

```markdown
# Loop: keep-ci-green

**Goal:** keep main's CI passing
**Stop condition:** latest CI run on main is green and Open is empty
**Cadence:** on push to main + daily at 09:00
**Human gate:** PR merge

## Open
- [ci] `test/auth.spec.ts` failing since a3f9c1 ("token refresh returns 401")

## Done
- [ci] flaky `db.pool` timeout: raised pool size, `npm test` green (run #4821)

## Blocked
- [ci] `e2e/checkout` failing: needs a staging secret only an admin can rotate
```

## 3. Discovery

Each cycle, the triage step reads the latest CI run for `main`. For every failing job it adds one item to `## Open`: the failing test, the first failing commit, and the error line. Passing jobs add nothing.

## 4. Maker and checker

- **`loop-maker`** takes the top `## Open` item, reproduces the failure locally, writes the smallest fix, and reports the command that proves it (`npm test -- test/auth.spec.ts`).
- **`loop-checker`** runs that command plus the full suite. Green means it opens a PR and moves the item to `## Done` with the run number. Red means it leaves the item in `## Open` with the new failure, or moves it to `## Blocked` when the fix needs access a human controls (the staging secret above).

## 5. Trigger

- A GitHub Action on push to `main` runs one cycle.
- A daily scheduled task at 09:00 catches anything the push-triggered run left open.

## 6. Why it stays safe

The maker never declares its own fix correct, the checker runs the real suite rather than trusting a summary, and no code reaches `main` without a person merging the PR. When the loop hits something it cannot verify, the item lands in `## Blocked` instead of shipping a guess.
