---
name: pr-comment-cleanup
description: On request, trim and remove code comments that the current PR adds or changes.
metadata:
  opencode/autoinvoke: false
---

# PR Comment Cleanup

## Scope

Work only on the comments in the current PR.

1. Find the base branch with `gh pr view --json baseRefName -q .baseRefName`. If there is no PR, use `main`.
2. Run `git fetch origin <base>`.
3. Get the PR changes with `git diff $(git merge-base origin/<base> HEAD)`. Also check `git status --short` for untracked files.
4. Change only comments that the PR adds or changes. Do not touch other comments or code.

## Trim

- Keep a comment only when the code cannot explain itself. Remove the other comments.
- Make the comments that you keep short.
- Remove comments that repeat the intent doc, the plan, or the PR description.
- Remove comments about temporary work, such as stress tests or local test steps.

Keep TODO comments. Keep magic comments and tool directives, such as shebang lines, encoding lines, type checker directives, and linter directives like `eslint-disable`.

## Finish

1. Do not commit or push.
2. Give a list of each removed or changed comment and its file.
