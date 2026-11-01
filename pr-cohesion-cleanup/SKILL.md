---
name: pr-cohesion-cleanup
description: On request, remove dead code that the current PR adds or leaves behind, and split large files that the PR adds or grows by responsibility.
metadata:
  opencode/autoinvoke: false
---

# PR Cohesion Cleanup

Each file has one responsibility, and each line of code has a caller.
Split a file by responsibility, not by line count.

## Scope

Work only on the code in the current PR.

1. Find the base branch with `gh pr view --json baseRefName -q .baseRefName`. If there is no PR, use `main`.
2. Run `git fetch origin <base>`.
3. Get the PR changes with `git diff $(git merge-base origin/<base> HEAD)`. Also check `git status --short` for untracked files.
4. Change code that the PR adds or changes, and code that has no callers because of the PR. Do not touch other code.

## Remove dead weight

- Functions, methods, constants, classes, types, and private helpers with no caller.
- Templates, components, view helpers, routes, and translation keys that nothing renders or calls.
- Branches that cannot run, for example a condition that is always true after the PR, or a `switch` branch for a value that no caller passes.
- Duplicate helpers. Keep one helper and move the callers to it.
- Abstractions that the PR makes obsolete, for example a wrapper with one caller left, or an option that every caller sets to the same value.
- Commented-out code, debug output, and breakpoints.
- Test fixtures, factories, and helpers that only the removed code used.
- Code that only tests call, when the PR adds both the code and the tests. Remove both. If the code existed before the PR, report it and do not change it.

If the project has a dead-code tool, run it to find candidates. Use only the results in files that the PR changes. These tools report false positives, for example code that a framework, a template, or reflection calls by name. Confirm each candidate with a search.

## Split large files

Look at each hand-written file that the PR adds, or grows by more than about 100 lines, when the result is about 500 lines or more. Do not apply this to generated code, type stub files, schema dumps, lock files, or fixtures.

If the file has more than one responsibility, split it with the layout rules in the nearest `AGENTS.md` or the project docs. If the project has no layout rules, follow the patterns of nearby files. Common splits:

- Display formatting in a model or a view goes to a presenter or a view helper.
- Business logic in a job, a script, or a request handler goes to the domain code.
- A request handler longer than about 10 lines moves its logic to the domain code.
- A large template or component goes into smaller parts.
- A class with separate responsibilities moves one of them to a new module, or to the class that owns the data.
- Tests move with the code. Each new file gets its own test file.

Do not split a cohesive file only because of its line count. Record the reason for each large file that you do not split.

Do not do a split that needs one of these. Report it and ask the user:

- a new top-level namespace or directory, because it can change code ownership, such as `CODEOWNERS`,
- a new layer or pattern that the project does not use yet,
- a new name or path for a class whose name stored data or a job queue holds. See `pr-cutover-cleanup`.

## Evidence for each change

Before you remove code, record:

1. The code and its location.
2. The search for callers. Search with `rg` or `git grep` for each form of the name, such as the identifier and the string. Search the source, config, scripts, migrations, templates, and tests. Include routes, dynamic calls, reflection, and lookups by name.
3. The helpers, tests, and fixtures that only this code uses.

Before you split a file, record:

1. The file, its line count, and each responsibility that you found.
2. Each new file and the responsibility that it holds.

If you cannot record an item, do not change the code. List it as not judged.

## Finish

1. Run the checks in the nearest `AGENTS.md` or the project docs:
   1. The tests for the changed code and for the split files.
   2. The type checker and the linter, if the project has them.
   3. If you removed a route or another item that a code generator reads, run that generator.
   4. `git diff --check`.
2. Do not commit or push.
3. Report:
   - each removed item and the result of its caller search,
   - each split, with the old file and the new files,
   - each split that needs approval from the user,
   - each large file that you did not split, and the reason,
   - code that only tests call and that existed before the PR,
   - each candidate that you did not have evidence to judge,
   - removed lines and moved lines, as separate counts.
