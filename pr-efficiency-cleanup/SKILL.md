---
name: pr-efficiency-cleanup
description: On request, fix avoidable work in the code that the current PR adds or changes, such as N+1 queries, repeated service calls, and new code that repeats an existing helper.
metadata:
  opencode/autoinvoke: false
---

# PR Efficiency Cleanup

Fix a cost only when you can name it and a real workload makes it matter.
Keep readable code when a faster form would not make a difference.

## Scope

Work only on the code in the current PR.

1. Find the base branch with `gh pr view --json baseRefName -q .baseRefName`. If there is no PR, use `main`.
2. Run `git fetch origin <base>`.
3. Get the PR changes with `git diff $(git merge-base origin/<base> HEAD)`. Also check `git status --short` for untracked files.
4. Change only code that the PR adds or changes. Do not touch other code.

## Fix: queries

- **N+1 queries.** A read of a related record in a loop, without eager loading. If the project has an N+1 query detector, treat each of its warnings as a defect.
- **Queries in a loop.** A count, an existence check, or a lookup for each item. Load all the items in one query and index them by ID, or use a grouped count.
- **Loads to answer a yes/no question.** A query that loads rows only to find if any exist. Use an existence query.
- **Count and load together.** Do not count rows with one query and then load the same rows with a second query. Count the loaded rows.
- **Full records when you need one column.** Select only that column. When the records are already loaded, use them to prevent a second query.
- **Row-by-row writes.** An insert or an update in a loop. Use a bulk insert, upsert, or update. Bulk writes often skip validations and callbacks. Keep row-by-row writes when these matter.
- **Table scans in scheduled jobs.** A loop over a full table in a scheduled job. Use batched iteration that can continue after an interruption. For work that cannot finish in one run, split it across runs.
- **Aggregates on large tables on each run.** A count, a sum, or a maximum in a scheduled job. Use a persisted counter or a summary row.

## Fix: code

- Use the collection functions of the language. For example, a filter followed by the first item becomes a find, a map followed by a flatten becomes a flat map, and a sort followed by the last item becomes a maximum.
- A loop that builds a lookup map becomes the index or group function of the language or the library.
- A value that one function or one loop computes more than once becomes a local variable, or a memoized value in the style of the file. Make sure that the memoization also keeps a null or `false` result.
- A serialization followed by a parse, only to copy or convert data.
- A deep copy of data that the code does not change.

## Fix: external calls

- The same call to another service more than once for the same data in one unit of work. Make the call once and pass the result.
- One call for each item when the client has a list or batch method.

## Reuse

Before you keep a new helper, search for code that already does the same work. Use the existing code. Look for:

- The existing client for each external service. Do not add a second client for the same service.
- Shared modules, mixins, and base classes.
- Shared services and helpers, for example in `lib/`, `utils/`, or `shared/`.
- Shared UI components and frontend helpers.
- Functions of the standard library and the framework.

Keep a new helper when the existing one behaves differently in a way that the PR needs. Record the difference.

## Report, do not change

- A new filter, sort, or join column with no index. The index migration ships in its own PR first.
- A call to another service in a request handler, a view, or a model getter. It usually belongs in a background job or a service.
- A new scheduled job. The PR description must give the table sizes and the expected runtime.

## Evidence for each change

Before you change code, record:

1. The code and its location.
2. The cost: the number of queries, calls, rows, or allocations, and the workload. For queries, use `EXPLAIN` on data of a realistic size, and the SQL log or the N+1 detector output from the tests.
3. The simpler form, or the existing helper and its location.
4. Behavior that the change can alter: callbacks, validations, order, or null handling.

If you cannot record the cost, do not change the code. List it as not judged.

## Finish

1. Run the checks in the nearest `AGENTS.md` or the project docs:
   1. The tests for the changed code. If the project has an N+1 query detector, look for its warnings in the output.
   2. The type checker and the linter, if the project has them.
   3. `git diff --check`.
2. Do not commit or push.
3. Report:
   - each change, with its cost before and after,
   - each new helper that you replaced with existing code,
   - each item from [Report, do not change](#report-do-not-change),
   - each candidate that you kept, and the reason,
   - each candidate that you did not have evidence to judge.
