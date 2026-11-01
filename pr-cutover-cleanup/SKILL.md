---
name: pr-cutover-cleanup
description: On request, remove aliases, dual paths, and transition code that the current PR keeps only for internal callers. Keep the paths that stored data, queued work, or external callers need.
metadata:
  opencode/autoinvoke: false
---

# PR Cutover Cleanup

Migrate the callers and remove the old path in the same PR.
Keep an old path only when stored data, queued work, or an external caller needs it.
"Old code still calls it" is not a reason to keep a path when you can migrate that code.

## Scope

Work only on the transition code in the current PR.

1. Find the base branch with `gh pr view --json baseRefName -q .baseRefName`. If there is no PR, use `main`.
2. Run `git fetch origin <base>`.
3. Get the PR changes with `git diff $(git merge-base origin/<base> HEAD)`. Also check `git status --short` for untracked files.
4. Change code that the PR adds or changes, and old code that the PR replaces. Do not touch other code.

## Remove

- **Aliases and forwarders.** An alias, a function that only calls its new name, a re-export, or a constant kept for an old name, when every caller is in this repository. Migrate the callers.
- **Dual paths for internal callers.** A branch on a flag, an environment variable, a config value, or an argument that selects the old or the new behavior, when only code in this repository selects the old path.
- **Old argument forms.** For example, a function that accepts both `cluster_id` and `cluster_name` when every caller passes `cluster_id`. For jobs, also check the queued-jobs rule in [Keep](#keep).
- **Comments about a compatibility path that you removed.**

Example: the PR renames `drain` to `mark_draining` and keeps `drain` as an alias. All callers are in this repository. Migrate them and remove the alias.

## Keep

The Keep list wins over the Remove list. Each item stores or sends a name or a shape that old data or another system still uses.

- **Names in stored data.** A database, a cache, or a file can store a class name or a function name. For example:
  - Job and workflow frameworks that store the class name, the arguments, and the results of a run. A retry can replay the prior steps, arguments, and results. If you rename or remove a step class, a step argument, or a result name, runs that are active, paused, or retried can fail.
  - Single-table inheritance `type` columns, polymorphic type columns, and serialized objects.
- **Queued jobs.** A job queue stores the job name and its arguments. Every queued job runs, and old and new code run at the same time during a deploy. Remove an old job, or an old argument form, only when both of these are true:
  1. A schedule or a reconciliation loop enqueues the work again, or a lost run causes no harm.
  2. The job is safe to run twice.
  If not, keep the old form for one deploy.
- **Stored values.** Enum values that rows still hold, and the old shape of serialized data. Keep the reader for the old shape until a data migration converts the rows.
- **Schema transitions.** Reads of an old column during a column move, and code that ignores a column before the column drop. The migration ships in its own PR first.
- **External contracts.** Routes, public APIs, GraphQL schemas, MCP tool names and parameters, message formats on queues and event streams, webhook and chat interaction payloads, CLI command and task names, and config that other systems read. Ask the user before you change one of these.

For each old path that you keep, make sure that a TODO states the condition for its removal, for example `TODO: Remove after all queued DrainJob runs finish.`

## Evidence for each removal

Before you remove an old path, record:

1. The old path and its location.
2. The search for callers. Search with `rg` or `git grep` for each form of the name, such as the identifier and the string. Search the source, config, scripts, migrations, templates, and tests. Include dynamic calls, reflection, and lookups by name.
3. Proof that no stored data, queued job, or external caller uses the old name or shape.
4. The failure that an old caller gets after the removal, and how many callers can get it.

For a name that a job or workflow framework stores, count the production runs that are not finished or that can be retried. If you cannot query production data, give the exact query to the user.

If you cannot record an item, keep the old path and list it as not judged.

## Finish

1. Run the checks in the nearest `AGENTS.md` or the project docs:
   1. The tests for the changed code and for the migrated callers.
   2. The type checker and the linter, if the project has them.
   3. If you removed a route or another item that a code generator reads, run that generator.
   4. `git diff --check`.
2. Do not commit or push.
3. Report:
   - each removed old path and the callers that you migrated,
   - each old path that you kept, the reason, and the condition for its removal,
   - each candidate that you did not have evidence to judge.
