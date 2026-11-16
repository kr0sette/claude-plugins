---
disable-model-invocation: true
description: Work only. Review someone else's PR, regressions first, and recommend approve or block. Pass a PR number, URL or branch. Never posts to GitHub.
---

# Review

Review the target I named: a PR number or URL, or a branch. If I named nothing, ask which one.
This is for reviewing **someone else's** code. Output findings only: no recap, no praise.

## Project rules

Read the repo's `CLAUDE.md` (or `AGENTS.md`) and `.claude/review.md` if present before reviewing.
Their rules add to these and win on a conflict.

## Backend companion review

If the PR is in a Pendo backend repo (`pendo-appengine`, or a backend microservice such as l10n or APPMS) or mostly changes `.go` files:
- Also run `/pendo-team-blob:review-guide-backend` on the same PR.
- Merge its findings into the one table below, removing duplicates.
- If it exits as "not applicable" or isn't installed, continue without it and say which in one line.

## Gather

1. PR title, body, labels, commits, full diff, CI status, and earlier reviews (don't ask to drop what an earlier review requested).
2. The ticket: find the key in the branch, title or body, fetch it via the available tracker connector, and read what it asked for.
3. Surrounding code, not just the hunks: callers of anything changed, and existing helpers that do the same thing.

## Priorities (highest first)

1. **Regressions to existing workflows.**
   - Shared code paths: confirm old flows behave as before.
   - Contracts: API shapes, stored formats, config defaults, deploy wiring.
2. **Redundancy and readability, especially for other agents.**
   - Duplicated logic or an existing helper that should be reused.
   - Unclear names, noisy or wrong comments, and logic a reader could misread.
   - Suggest a consolidation or more scalable shape only when it's small, needs no big refactor, and doesn't derail the card's goal.
3. **Riders and scope.** Changes the ticket doesn't need:
   - Riders that alter existing flows: recommend **drop**.
   - Standalone new work: recommend **split** into its own PR.
4. **Ticket fit.** Acceptance criteria that are missing or only partly done.
5. **Correctness and security.** Edge cases, swallowed errors, races, leaks, injection, missing authorization.
6. **Accessibility** (frontend files only). Accessible names, labels, keyboard use, focus, color-only meaning.
7. **Tests (absolute last).** Missing or weak coverage goes in the final rows.

Don't run tests or lint; CI covers them.

## Verify before reporting

- Try to disprove every finding: re-read the surrounding code and confirm the failure is reachable.
- If it can't be disproved but you couldn't confirm it (external system, real data), mark the issue "Unverified" and say what to check.
- If you can't name a concrete failure, drop it. That includes readability nits with no real way to mislead a reader or agent.

## Output

One table, most critical to least critical, tests last. Terse; lead with the fix.

| # | Severity | File:line | Issue | Why it matters | Suggested fix |
| --- | --- | --- | --- | --- | --- |

Severity is one of:
- **Critical:** would cause a regression, bug, security hole, data loss, or a PR that misses the ticket. Blocks merge.
- **Suggestion:** improves quality but isn't a failure: duplication, readability for agents, riders to drop or split, accessibility gaps.
- **Nit:** minor and optional, but still real. Keep these few, and put them just above the test rows.

- **Numbering:** number rows 1, 2, 3... in order so I can refer to them as #1, #2.
- **Findings:** each row is one finding. Combine repeats across call sites into one row.
- **Unmet criteria:** add a row per missing or partial acceptance criterion.
- **No recap:** don't list what you checked or found fine, and don't summarize the PR.
- **No findings:** say "No changes needed." and nothing else.
- **Verdict:** end with one line: `Block` if any row is Critical, otherwise `Approve`.

## Comments I'd add

After the table, list the review comments I would post, as a second table:

| # | File:line | Comment |
| --- | --- | --- |

- Use the same # as the finding it comes from.
- Only lines inside the diff.
- Terse, solution-focused, no commit hashes.

## Rules

- **Draft only.** Never post, approve, comment or resolve on GitHub unless I explicitly say so.
- **Stay in scope.** Don't flag pre-existing issues unrelated to the change.
- **Terse.** Plain words, no filler.

## Edge cases

- **No diff:** say so and check the target.
- **Large diff (over 20 files or 1000 lines):** group by concern, go deep on business logic, and suggest splitting.
- **Generated or binary files:** skip, but flag a generated file that looks hand-edited.
- **`gh` missing or unauthenticated:** say so and ask for the diff.
