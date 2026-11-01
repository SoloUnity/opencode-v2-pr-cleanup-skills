---
name: pr-naming-cleanup
description: On request, rename variables that the current PR adds or changes so they are human readable and descriptive.
metadata:
  opencode/autoinvoke: false
---

# PR Naming Cleanup

## Scope

Work only on the variables in the current PR.

1. Find the base branch with `gh pr view --json baseRefName -q .baseRefName`. If there is no PR, use `main`.
2. Run `git fetch origin <base>`.
3. Get the PR changes with `git diff $(git merge-base origin/<base> HEAD)`. Also check `git status --short` for untracked files.
4. Rename only variables that the PR adds or changes. Do not touch other names.

## Rename

- Make each variable human readable and descriptive. A reader must understand it on first read, without other context.
- Replace abbreviations, single letters, and general names such as `data`, `result`, `obj`, `tmp`, or `x`.
- Use the naming conventions of the repository.
- Update all references to a renamed variable.

## Finish

1. Run the tests for the changed files.
2. Do not commit or push.
3. Give a table of each old name, new name, and file.
