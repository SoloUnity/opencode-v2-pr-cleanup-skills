---
name: pr-docs-cleanup
description: On request, rewrite the documentation that the current PR adds or changes in ASD-STE100 Simplified Technical English. The text becomes clear and short, and it keeps all of its meaning.
metadata:
  opencode/autoinvoke: false
---

# PR Docs Cleanup

Write documentation that a reader understands on first read.
Use ASD-STE100 Simplified Technical English. Keep all of the meaning.
If a simpler sentence loses a fact, keep the old sentence.

## Scope

Work only on the documentation in the current PR.

1. Find the base branch with `gh pr view --json baseRefName -q .baseRefName`. If there is no PR, use `main`.
2. Run `git fetch origin <base>`.
3. Get the PR changes with `git diff $(git merge-base origin/<base> HEAD)`. Also check `git status --short` for untracked files.
4. Change only the sentences that the PR adds or changes. In a new file, change all of the text. Do not touch other text or code.

Documentation is a file that people read and that no code reads. For example: Markdown and text files in `docs/`, design and intent docs, `README.md`, `AGENTS.md`, `SKILL.md` and its reference files, and `CHANGELOG.md`.

These are not documentation. Do not change them:

- Code comments. `pr-comment-cleanup` owns them.
- Files that code reads, such as LLM prompts that code loads, templates, and test fixtures.
- Generated docs, for example files that a script builds, and files with a "generated" or "do not edit" header. Report the change for the source file.
- UI text, translation values, and error messages.

## Words

- Use simple, common words. Give each word one meaning. Use the same word for the same thing each time.
- Use a technical term only for a specific item: a class, function, variable, config key, file path, command, table, or product, such as `UserAccount`, `cache.enabled`, Kafka, or PostgreSQL. Put code items in backticks.
- Replace other technical terms and jargon with plain words. For example, "idempotent" becomes "safe to run more than one time", "leverage" becomes "use", and "in order to" becomes "to".
- If no plain words keep the full meaning, keep the term. Explain it the first time that you use it.
- Use one verb, not a phrasal verb: "find", not "find out".
- Do not use the "-ing" form of a verb, except in a name or a code item.
- Do not use more than three nouns in a row. Break the group with "of", "for", or "in".
- Do not remove "the", "a", or "this" to make a sentence shorter.
- Remove filler, such as "basically", "simply", "note that", and "it is important to".

## Sentences

- Write about one topic in each sentence.
- Write instructions in 20 words or fewer, and descriptions in 25 words or fewer.
- Use the active voice. Say who or what does the action.
- Use the present tense for facts. Use the past or future tense only when the time is important.
- Write instructions as commands, with one action in each sentence. Put the condition first: "If X, do Y."
- Put a warning before the step that it applies to.
- Write about one topic in each paragraph, in six sentences or fewer.
- Use a list for steps, options, and groups of conditions. Number the steps.
- Say each fact one time in a file. Remove a sentence that repeats a fact.

## Keep

The Keep list wins over all other rules.

- Each fact: each statement, condition, exception, limit, number, unit, default, name, reason, warning, and example.
- The strength of each word, such as "must", "should", "can", "never", "always", "only", and "at least". Do not change "should" to "must", or "must" to "can".
- Words that give a real doubt or a real limit, such as "probably" or "for most users". Remove only filler.
- These items, with no change: code blocks, inline code, commands, file paths, URLs, links, diagrams, quoted UI text, error messages, log lines, and quotes from people or other documents.
- The front matter.
- A heading that a link points to. Search for `<file>#<anchor>` and `](#<anchor>)` in the repository.

## Evidence for each change

Before you change a file, record the search that shows that no code reads it. Search with `rg` or `git grep` for the file name in the source, config, scripts, and tests.

Make one change for each paragraph, list, or table. Before you change text, record:

1. The file, the lines, and the old text.
2. The new text.
3. The rule for each change, for example "sentence of 34 words" or "jargon: idempotent".
4. Each fact in the old text, and where it is in the new text.
5. For a changed heading: the search for links to its anchor.

If you cannot record an item, or a fact has no place in the new text, do not change the text. List it as not judged.

## Edit

1. Before you edit a file, save a copy in the session temporary directory.
2. Replace the old text with the new text.

## Finish

1. Compare each copy with its file. Each inline code item, link, URL, and number in the copy must also be in the file. A line that starts with `<` is a lost item. Put its fact back.

   ```sh
   items() { rg -o -N --no-filename '`[^`]+`|\]\([^)]+\)|https?://[^ )>]*[^ )>.,;:]|[0-9][0-9.,:%]*' "$1" | sort -u; }
   diff <(items <copy>) <(items <file>)
   ```

2. Run `git diff --check`.
3. Do not commit or push.
4. Report:
   - each changed paragraph, list, or table, with its file and its rules,
   - each technical term that you kept, and the reason,
   - each generated doc that needs a change in its source,
   - each candidate that you did not have evidence to judge.
