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

1. PR title, body, labels, commits, full diff, and CI status.
2. **Every existing comment, not just the review summaries.** Fetch all three:
   - Inline threads, with resolved state: `gh api graphql` on `pullRequest.reviewThreads` (`isResolved`, `isOutdated`, `path`, `line`, `comments`).
   - Review summaries: `gh pr view <n> --json reviews`.
   - General PR comments: `gh pr view <n> --json comments`.
3. **The ticket (required, don't skip):** find the key in the branch, title or body.
   - If the tracker tools are deferred, load them with ToolSearch (query "jira get issue"), then fetch the ticket. Jira may also need the cloud ID from the accessible-resources tool first.
   - Read the description and acceptance criteria.
   - Only if the fetch truly fails, say so in one line at the top and continue.
4. Surrounding code, not just the hunks: callers of anything changed, and existing helpers that do the same thing.
5. **Read code at the PR's head commit, never the local checkout** (it is often on `master` and has different line numbers).
   - Get the head SHA: `gh pr view <n> --json headRefOid`.
   - Read a file at that SHA: `gh api "repos/<owner>/<repo>/contents/<path>?ref=<sha>" --jq .content | base64 -d`, or `git show <sha>:<path>` after fetching.
   - Only use the local files if the checkout is already on the PR's head commit.
   - Every `File:line` must come from the PR head version, or from the diff's new-file line numbers.

## Priorities (highest first)

1. **Architecture and approach.** Judge the overall design before the details.
   - Is this the right approach for the goal, or is there a simpler or more fitting one?
   - What else does it touch? Find other areas, inside and outside the card's scope, that could break or behave differently: shared code, callers, other services, config, data formats.
   - Does it lay a good foundation for scaling, or does it box in future work (hardcoded cases, one-off paths, hidden coupling)?
2. **Regressions to existing workflows.**
   - Shared code paths: confirm old flows behave as before.
   - Contracts: API shapes, stored formats, config defaults, deploy wiring.
3. **Redundancy and readability, especially for other agents.**
   - Duplicated logic or an existing helper that should be reused.
   - Unclear names, noisy or wrong comments, and logic a reader could misread.
   - Suggest a consolidation or more scalable shape only when it's small, needs no big refactor, and doesn't derail the card's goal.
4. **Riders and scope.** Changes the ticket doesn't need:
   - Riders that alter existing flows: recommend **drop**.
   - Standalone new work: recommend **split** into its own PR.
5. **Ticket fit.** Acceptance criteria that are missing or only partly done.
6. **Correctness and security.** Edge cases, swallowed errors, races, leaks, injection, missing authorization.
7. **Accessibility** (frontend files only). Accessible names, labels, keyboard use, focus, color-only meaning.
8. **Tests (absolute last).** Missing or weak coverage goes in the final rows.

Don't run tests or lint; CI covers them.

## Existing comments

Before writing any finding, compare it to every existing comment and thread.
- **Never repeat a point already made**, whether the thread is open, resolved or outdated. Skip it, or reference it by thread if my take adds something new.
- **Include bot and human comments.** Both count as existing comments.
- **Only check the Critical ones** (a regression, bug, security hole or ticket miss).
  - Re-read the code at the PR head and decide: **Valid** or **Not valid**, with a one-line reason.
  - If the author already fixed it, say **Addressed**.
  - An open, valid Critical comment counts toward the verdict.
- **Non-critical comments:** don't check or list them. Just don't repeat their points.

## Verify before reporting

Try to disprove every finding: re-read the surrounding code and confirm the failure is reachable.
- If you confirm it, it's a finding and goes in the main table.
- If it hinges on something you couldn't check (an external system's behavior, real data, infra outside the repo), it goes in the **Please verify** table. Say what to check and what breaks if it's wrong.
- If you can't name a concrete failure either way, drop it. That includes readability nits with no real way to mislead a reader or agent.

## Output

One table, most critical to least critical, tests last. Terse; lead with the fix.

| # | Severity | File:line | Issue | Why it matters | Suggested fix |
| --- | --- | --- | --- | --- | --- |

Severity is one of:
- **Critical:** would cause a regression, bug, security hole, data loss, or a PR that misses the ticket. Blocks merge.
- **Suggestion:** improves quality but isn't a failure: duplication, readability for agents, riders to drop or split, accessibility gaps.
- **Nit:** minor and optional, but still real. Keep these few, and put them just above the test rows.

- **Numbering:** number rows 1, 2, 3... in order so I can refer to them as #1, #2.
- **Findings:** each row is one confirmed finding. Combine repeats across call sites into one row.
- **Severity cell:** exactly one word: Critical, Suggestion or Nit.
- **Unmet criteria:** add a row per missing or partial acceptance criterion.
- **No recap:** never write a "checked and fine" or "no issues in..." list, in any form, and don't summarize the PR.
- **Architecture line:** start the output with one line: `Architecture:` followed by the main concern, or "no concerns". Then the table.
- **Closing line:** don't say anything about posting, drafts or what you did not do.
- **No findings:** say "No changes needed." and nothing else.
- **Verdict:** goes at the very bottom of the whole output (see "Verdict" below).

## Please verify

After the main table, list concerns you couldn't confirm, most costly first. Leave this section out if there are none. Number them as continuing rows (#N+1...).

| # | File:line | What to check | What breaks if it's wrong |
| --- | --- | --- | --- |

## Existing comments checked

After Please verify, list only the Critical existing comments (bot or human) with my validity check. Leave this section out if there are none.

| Thread (author, File:line) | Status | Verdict | Reason |
| --- | --- | --- | --- |

- **Status:** open, resolved or outdated.
- **Verdict:** Valid, Not valid or Addressed.
- Don't list non-critical comments, bot commands or status messages.

## Comments I'd add

After the table, list the review comments I would post, as a second table:

| # | File:line | Comment |
| --- | --- | --- |

- Use the same # as the finding it comes from.
- Only lines inside the diff.
- Terse, solution-focused, no commit hashes.

## Verdict

The last line of the output, after every table: `Block` if any row is Critical or a Critical existing comment is still open, otherwise `Approve`. Nothing comes after it.

## Rules

- **Draft only.** Never post, approve, comment or resolve on GitHub unless I explicitly say so. Don't mention this in the output.
- **Stay in scope.** Don't flag pre-existing issues unrelated to the change.
- **Terse.** Plain words, no filler.

## Edge cases

- **No diff:** say so and check the target.
- **Large diff (over 20 files or 1000 lines):** group by concern, go deep on business logic, and suggest splitting.
- **Generated or binary files:** skip, but flag a generated file that looks hand-edited.
- **`gh` missing or unauthenticated:** say so and ask for the diff.
