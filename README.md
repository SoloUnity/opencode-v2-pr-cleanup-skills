# opencode-v2-pr-cleanup-skills

OpenCode V2 skills that clean up the code in the current PR.

| Skill | What it does |
| --- | --- |
| `pr-cleanup` | Runs all the skills below, in order. |
| `pr-cutover-cleanup` | Removes aliases and old paths that only internal callers use. |
| `pr-cohesion-cleanup` | Removes dead code and splits large files. |
| `pr-defensive-cleanup` | Removes guards for states that cannot occur. |
| `pr-efficiency-cleanup` | Fixes N+1 queries, repeated calls, and duplicate helpers. |
| `pr-test-cleanup` | Removes tautological and duplicate tests. |
| `pr-naming-cleanup` | Gives variables descriptive names. |
| `pr-comment-cleanup` | Trims code comments. |
| `pr-docs-cleanup` | Rewrites docs in Simplified Technical English. |

## Install

```sh
git clone https://github.com/SoloUnity/opencode-v2-pr-cleanup-skills.git \
  ~/.config/opencode/skills/opencode-v2-pr-cleanup-skills
```

OpenCode finds the skills on the next run. To update, run `git pull` in that directory.

## Use

The skills do not load automatically. Ask for one by name, for example `@pr-cleanup`.
The skills change files but do not commit or push. They need `git` and the [GitHub CLI](https://cli.github.com/) (`gh`).
