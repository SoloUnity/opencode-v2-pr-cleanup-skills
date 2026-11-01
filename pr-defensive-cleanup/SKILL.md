---
name: pr-defensive-cleanup
description: On request, remove guards, catches, and fallbacks that the current PR adds for states that cannot occur, with evidence for each removal.
metadata:
  opencode/autoinvoke: false
---

# PR Defensive Cleanup

A guard stays when a credible input or state makes it fire.
Internal code trusts its contracts and fails loudly when a contract breaks.
Do not remove a guard only because it looks defensive.

## Scope

Work only on the guards in the current PR.

1. Find the base branch with `gh pr view --json baseRefName -q .baseRefName`. If there is no PR, use `main`.
2. Run `git fetch origin <base>`.
3. Get the PR changes with `git diff $(git merge-base origin/<base> HEAD)`. Also check `git status --short` for untracked files.
4. Change only guards that the PR adds or changes. Do not touch other code.

## Remove

These patterns are candidates. Remove one only when you have the evidence in [Evidence for each removal](#evidence-for-each-removal).

- **Null checks on values that cannot be null.** Optional chaining, a null check, an early return, or `x || default` on a value whose type, schema, or `NOT NULL` column excludes null.
- **Type checks that the type system already makes.** A runtime type check, a cast, or a conversion, such as a wrap in a list or a conversion to a string, on a value of a known type. Narrowing a union type is not defensive.
- **Defaults on required data.** A map lookup with a default, where the value must be present. Use a lookup that fails when the key is missing.
- **Lookups that hide a missing record.** A lookup that returns null, followed by a null check, or a catch of a not-found error that returns null, where the record must exist. Use the lookup that raises an error.
- **Catches that turn bugs into defaults.** A broad catch that logs, reports to an error tracker, or returns null, an empty value, or `false`, and then continues. In most job and request frameworks, an unhandled error already goes to the error tracker. Check this for the project. For jobs and workflows, use the retry and discard rules of the framework, with specific error classes.
- **Fallbacks for closed sets.** A default branch in a `switch` on an enum that returns a default value. Use an exhaustiveness check, or raise an error.
- **Warnings instead of errors.** A log line for a state that is a bug, followed by `continue` or `return`. Raise an error instead.
- **Repeated validation.** A check that a model validation, a database constraint, request validation, or an earlier guard in the same call path already makes.
- **Speculative retries, caches, and locks.** A retry loop around a call to another service when the job or the workflow already retries. Leave retry policy to jobs and workflows. A cache or a lock for a race that you cannot show can occur.
- **Checks for modules that always load.** For example, a check that a required library is defined before you use it.

## Keep

The Keep list wins over the Remove list.

- Validation of data from a boundary: request parameters, API and tool inputs, queue and event messages, webhook payloads, responses from other services, files that people edit, and serialized columns.
- Checks on values that can be null: nullable types, optional associations, and nullable columns.
- Idempotency handling. In reconciliation loops, workflow steps, and retried jobs, "already exists" and "already absent" are success, and observe-before-write is necessary.
- The per-message catch that reports the error in a queue consumer, and a top-level catch that reports the error and raises it again.
- A catch of an expected time limit in batch work that continues in the next run.
- Authorization checks, production guards, and concurrency limits.
- A guard that a bug fix added, with a test that fails without it.

## When the type does not prove the invariant

A type is proof only when the compiler or a runtime check enforces it. Do not trust the type in these cases:

- The project turns off type checks for this code.
- The value comes through an untyped value, an unchecked cast, an untyped file, or a function with no type signature.
- The value comes from job arguments, parsed JSON or YAML, or a map with string keys.
- The record is not saved. A `NOT NULL` column can be null before the save.
- Only an application validation enforces the value. Rows from before the validation can break it.

## Repair, do not remove

- **A necessary guard that uses a bare non-null assertion.** Replace it with an explicit check that raises a clear error.
- **A broad catch at a real boundary.** Narrow it to the error classes that the boundary raises. Let the other errors propagate.
- **A test that asserts a removed fallback.** Change the test to assert the error. If the test shows that the value comes from a boundary, keep the guard and the test.

## Evidence for each removal

Before you remove a guard, record:

1. The guard and its location.
2. The invariant that makes the guard unnecessary: the type, the column constraint, the upstream guard, or the framework behavior, and its location.
3. The callers that you checked. Search with `rg` or `git grep`. Include dynamic calls, templates, and config.
4. The error that the guard hides, and where that error goes after the removal.

If you cannot record an item, keep the guard and list it as not judged.

## Finish

1. Run the checks in the nearest `AGENTS.md` or the project docs:
   1. The tests for the changed code.
   2. The type checker and the linter, if the project has them.
   3. `git diff --check`.
2. Do not commit or push.
3. Report:
   - each removed guard, its invariant, and the error that now surfaces,
   - each repaired guard,
   - each candidate that you kept, and the reason,
   - each candidate that you did not have evidence to judge.
