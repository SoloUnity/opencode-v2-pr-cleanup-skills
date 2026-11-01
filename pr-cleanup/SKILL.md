---
name: pr-cleanup
description: On request, run all PR cleanup skills on the current PR. Read-only explore subagents find the facts for each skill, then the caller checks the facts and applies the changes in a fixed order.
metadata:
  opencode/autoinvoke: false
---

# PR Cleanup

Run the eight PR cleanup skills on the current PR.
The subagents find facts and do not change files. The caller checks the facts and makes all the changes.

## Skills

Apply the skills in this order. Earlier skills remove code, so later skills have less code to change. Names, comments, and docs come last.

The skill files are in the same skills directory as this skill.

| Order | Skill | Prefix | File |
|---|---|---|---|
| 1 | `pr-cutover-cleanup` | `CUTOVER` | `../pr-cutover-cleanup/SKILL.md` |
| 2 | `pr-cohesion-cleanup` | `COHESION` | `../pr-cohesion-cleanup/SKILL.md` |
| 3 | `pr-defensive-cleanup` | `DEFENSIVE` | `../pr-defensive-cleanup/SKILL.md` |
| 4 | `pr-efficiency-cleanup` | `EFFICIENCY` | `../pr-efficiency-cleanup/SKILL.md` |
| 5 | `pr-test-cleanup` | `TEST` | `../pr-test-cleanup/SKILL.md` |
| 6 | `pr-naming-cleanup` | `NAMING` | `../pr-naming-cleanup/SKILL.md` |
| 7 | `pr-comment-cleanup` | `COMMENT` | `../pr-comment-cleanup/SKILL.md` |
| 8 | `pr-docs-cleanup` | `DOCS` | `../pr-docs-cleanup/SKILL.md` |

## 1. Prepare one packet

Do these steps once, before you start the subagents.

1. Read the eight skill files.
2. Find the base branch with `gh pr view --json baseRefName -q .baseRefName`. If there is no PR, use `main`.
3. Run `git fetch origin <base>`.
4. Record the repository path, `git rev-parse HEAD`, and the merge base from `git merge-base origin/<base> HEAD`.
5. Save the PR changes to a file in the session temporary directory: `git diff <merge-base> > <tmp>/pr-cleanup.diff`.
6. Record the changed files from `git diff --name-only <merge-base>`, and the untracked files from `git status --short`.

Do not change the working tree until all subagents finish.

## 2. Start the fact finders

Start one subagent for each skill, all at the same time. Run them in the background if your tools can, with a new session for each one. Use a read-only explore agent: `explorer` under OMO Slim, `explore` in other setups.

Give each subagent:

- the packet from step 1,
- the full text of its skill file,
- its prefix,
- the [Fact finder instructions](#fact-finder-instructions).

Wait for all subagents to finish. Do not poll.
If a subagent fails, record the gap. Do not apply that skill, and continue with the others.

## Fact finder instructions

Put this section into each subagent task.

You are a read-only fact finder for one PR cleanup skill.

- Use the skill's Scope with the packet. Use the diff file and the commit IDs in the packet. Do not fetch or make a new diff.
- Use the skill's rules and its evidence list to find candidates. Do not do its edit steps or its Finish steps.
- Do not edit or write files. Do not commit, push, fetch, or install. Do not run tests, builds, or linters. Do not start subagents or load skills.
- Run only shell commands that do not change files, git state, or a database. For example: `git show`, `git grep`, `git log`, `rg`, a dead-code report, and an `EXPLAIN` of a query.
- If a check needs access that you do not have, such as a query on production data, give the exact check to the caller.
- A clean result is correct when you find no candidates. Do not invent candidates.

For each candidate, report:

1. An ID with your prefix, for example `DEFENSIVE-001`.
2. The file and line in the packet revision.
3. The action: `remove`, `repair`, `change`, `split`, `report only`, `keep`, or `not judged`.
4. Each evidence item that the skill requires, with file and line.
5. For `remove`, `repair`, `change`, and `split`: the exact change, as the old code and the new code.
6. The checks that the caller must do.

End with the scope that you covered, the files that you did not check, and all checks for the caller.

## 3. Check and apply

1. Make sure that `HEAD` and the diff are the same as in the packet. If they changed, report all facts as stale and stop.
2. Do the checks that the subagents gave to you, for example the stored-runs query in `pr-cutover-cleanup`.
3. Apply the skills in the order of the [Skills](#skills) table. For each candidate:
   1. Read the current code. An earlier skill can have changed or removed it. If the code is gone, skip the candidate and record it.
   2. Check the evidence against the code. If an evidence item is missing or wrong, do not change the code. List the candidate as not judged.
   3. Make the change that the skill describes. Obey the skill's Scope and Keep list.
4. When two skills change the same lines, the earlier skill wins. Check the later candidate again against the new code.
5. Change only code that a subagent reported.
6. Do not change `report only` items, splits that need approval, or external contracts. Report them to the user.

If you cannot edit files, give the confirmed changes to an agent that can, such as `fixer`.

## 4. Finish

1. Run the checks once, after all changes. Use the checks in the nearest `AGENTS.md` or the project docs:
   1. The tests for the changed code, the migrated callers, and the split files. If the project has an N+1 query detector, look for its warnings in the output.
   2. The type checker and the linter, if the project has them.
   3. If a change removed a route or another item that a code generator reads, run that generator.
   4. `git diff --check`.
   5. If you applied a `pr-docs-cleanup` change, compare each doc copy with its file, as its Finish step shows.
2. If a check fails, find the change that caused it. Fix it or revert it, and record that.
3. Do not commit or push.
4. Report:
   - one section for each skill, in order, with the items that its Finish step lists and the candidate IDs,
   - the candidates that you skipped because their code was gone,
   - the conflicts between skills, and how you resolved them,
   - the items that the user must decide,
   - the checks that you ran and their results,
   - each subagent that failed or did not cover its full scope.
