---
name: pr-review-checklist
description: Reviews a Go diff against house rules - error wrapping, table tests, no naked returns. Use when asked to review a PR, check staged changes, or before pushing.
---

# PR Review Checklist

Review a Go diff against our house rules and report what to fix.

## Instructions

1. Read the staged diff (`git diff --staged`).
2. Flag any error not wrapped with `%w`.
3. Require a table test for every new exported function.
4. Ignore anything under `vendor/`.
5. Report findings as a ranked list, most severe first.

## Examples

User says: "review my staged changes"

1. Read the diff.
2. Apply the four rules.
3. Return the ranked list.

## Troubleshooting

**Nothing staged**
Cause: no `git add` yet.
Fix: stage the changes, then re-run.
