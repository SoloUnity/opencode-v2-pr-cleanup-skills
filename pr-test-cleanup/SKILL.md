---
name: pr-test-cleanup
description: On request, remove tautological, duplicate, and low-value tests that the current PR adds or changes, with evidence for each removal.
metadata:
  opencode/autoinvoke: false
---

# PR Test Cleanup

A test stays when a credible regression makes it fail and no stronger test already catches that regression.
Optimize for confidence, not for the number of removed tests.

## Scope

Work only on the tests in the current PR.

1. Find the base branch with `gh pr view --json baseRefName -q .baseRefName`. If there is no PR, use `main`.
2. Run `git fetch origin <base>`.
3. Get the PR changes with `git diff $(git merge-base origin/<base> HEAD)`. Also check `git status --short` for untracked files.
4. Change only tests that the PR adds or changes. Do not touch other tests or production code. The one exception is in [Edit](#edit).

## Remove

- **Tautological tests.** These tests repeat the implementation, or pass even when the production code is wrong.
- **Copied expected values.** The test restates a constant, a queue name, a superclass, an association, or a literal list from the source. It fails only when someone edits that line on purpose.
- **Asserted stubs.** The test stubs a function, a fixture, or a translation lookup, then asserts the value that the stub returned.
- **Duplicate tests.** The test sends the same input to the same contract as another test, often in another file, or a subclass test repeats the base class test. If the other test is outside the PR, remove the PR test.
- **Private call-shape tests.** The test calls a private function directly, for example with reflection, although a test through the public entry point covers the same branch.
- **Misplaced tests.** The test file names one class or module, but the test exercises another one that has its own test file.
- **Tests that do not add coverage.**
- **Tests that only check prompts, config values, names, UI strings, or log output.** This includes a test that loads a config file and asserts the value that the file sets.

## Keep

The Keep list wins over the Remove list.

Keep a test if it is the only coverage for a behavior. Also keep a test that is the independent guard of one of these, even when it looks like a copy of the source:

- a wire format, serializer output, tool schema, or telemetry event shape that another system reads,
- an authorization, permission, or tool-allowlist boundary,
- a persisted format, storage key, or migration behavior,
- a feature activation or flag-gating condition,
- observable call ordering, and
- a regression with a credible failure mode, such as a guard added by a fix.

## Repair, do not remove

- **Assertions that cannot fail.** For example, an assertion that a value is truthy where an equality check was meant, or a null check on a function that always returns null. Make the assertion correct.
- **Failing tests.** A failing test can show a product bug. Reproduce the failure and report it. Do not remove the test.

## Evidence for each removal

Before you remove a test, record:

1. The test name and its location.
2. The failure that the test can detect.
3. The stronger test that remains, or the reason that none is necessary.
4. The helpers, constants, fixtures, and setup state that only this test uses.

If you cannot record an item, keep the test and list it as not judged.

**Production code that only tests call.** Do not change it. Report it with the result of a search for callers outside the tests, in the source, config, scripts, migrations, views, components, and templates. A function with a common name, or one that reflection or a template calls, can have callers that a name search misses.

## Edit

- Remove the helpers, constants, fixtures, and setup state that only the removed tests use.
- When a removed test held the only copy of an assertion, move that assertion to the owning test file, also when that file is outside the PR.
- Do not add a replacement test that restates the same implementation.

## Finish

1. Run the checks in the nearest `AGENTS.md` or the project docs:
   1. The changed test files, and the test files that cover the same production code.
   2. The type checker and the linter, if the project has them.
   3. `git diff --check`.
2. Do not commit or push.
3. Report:
   - each removed test, its reason, and the stronger test that remains,
   - each repaired assertion,
   - each candidate that you kept, and the reason,
   - each candidate that you did not have evidence to judge,
   - production code that only tests call,
   - removed test lines and changed production lines, as separate counts.
