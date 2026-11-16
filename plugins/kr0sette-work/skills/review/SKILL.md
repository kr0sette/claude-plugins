---
disable-model-invocation: true
description: Work only. Review my current changes (or a PR number/branch) before I ship them. Use when I say "review", "review my changes", or "review PR 123".
---

Review the code changes for me. Target: the argument if given (PR number or branch), otherwise the current uncommitted diff plus commits ahead of the base branch.

1. Get the diff (`git diff`, or `gh pr diff <n>` for a PR). Read surrounding code when the diff alone isn't enough.
2. Look for, in this order: bugs and broken edge cases, missing or weak tests, unclear naming or logic, needless complexity, anything risky (security, data loss, perf).
3. Only report what you verified in the code. Label guesses as "unverified".

Output, concise:
- **Verdict**: ready / needs changes, one line.
- **Must fix**: `file:line` — problem — why it matters.
- **Consider**: smaller suggestions.
- **Good**: one line on what's solid.

Explain the "why" behind each finding in plain language. Do not post comments to GitHub or edit files unless I ask.
