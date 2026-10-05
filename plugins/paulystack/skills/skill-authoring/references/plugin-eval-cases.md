# `claude plugin eval` case format

Requires Claude Code v2.1.269+. Full docs: code.claude.com/docs/en/plugin-evals

## Contents
- [Layout](#layout)
- [prompt.md](#promptmd)
- [case.yaml](#caseyaml)
- [fixture.sh](#fixturesh)
- [Graders](#graders)
- [Run](#run)
- [User-invoked skills](#user-invoked-skills)

## Layout

```
<plugin-root>/
└── evals/
    ├── <skill>/            # group directory, not itself a case
    │   └── <case>/
    │       ├── prompt.md   # frontmatter = run settings, body = the user's message
    │       ├── case.yaml   # optional; run settings the prompt can't express
    │       ├── fixture.sh  # optional; builds the workspace before the run
    │       └── graders/    # one grader per .md file
    └── results/            # written by runs; gitignore it
```

## prompt.md

```markdown
---
description: One line, for humans reading the suite.
tags: [my-skill]
max_turns: 30
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Write, Edit]
---

A realistic request, phrased the way a user types it, without naming the skill.
```

- `@path` isn't expanded, and the workspace starts empty unless a `fixture.sh` fills it — so either inline every fact the run needs, or build the state with a scaffold.
- Runs can't ask questions. If the skill normally interviews the user, say "don't ask me questions".
- `plugins` defaults to the nearest enclosing plugin. Only if the summary shows no `W/OUT` column, set it explicitly: `plugins: ["../../.."]` (the path from the case to the plugin root).

## case.yaml

Optional, for settings `prompt.md` frontmatter can't express — chiefly a scaffold. Where both set a field, `prompt.md` wins.

```yaml
schema_version: "1.1"     # required
name: small-diff-text-only
context:
  scaffold_script: fixture.sh
```

Also takes `description`, `tags`, `plugins`, `runs`, `expected_outcome`; an `execution:` block (`model`, `max_turns`, `timeout_seconds`, `allowed_tools`, `append_system_prompt`, `env`); and under `context:`, `history_file` (a `.jsonl` transcript to start from) and `add_dirs` (read-only fixture directories).

## fixture.sh

Runs before Claude starts, in the empty workspace and outside the agent's sandbox — so `git` works here even though it fails inside a run. Its stdout isn't graded. Make it executable.

```bash
git init -q -b main
printf 'timeout = 30\n' > src/config.py
git -c user.name=eval -c user.email=eval@x commit -qam baseline
sed -i '' 's/30/60/' src/config.py    # the uncommitted change under test
```

Grade a scaffolded file by its contents, not `file_exists` — that grader counts only files created *during* the run.

## Graders

| type | options | passes when |
|---|---|---|
| `regex` | `pattern`, `flags`, `match` (`not_contains`, `count:N`), `target` | JavaScript regex found in target |
| `tool_used` | `tool`, `input_match`, `min`, `max` | matching call count within min–max |
| `tool_order` | `before`, `after` | first `before` call precedes first `after` call |
| `file_exists` | `path` (glob), `exists` | a file created during the run matches |
| `llm` | body = criteria, `focus` | judge votes PASS on the rubric |

- Common keys: `weight` (default 1), `arm: with-only | both`. Use `flags: i`, not `(?i)`.
- `target` / `focus` values: `last_message` (default), `trace`, `files` (created paths only), `{ source: file, path: <path> }` (file contents).

Skill fired. As written this is an indicator, not scored in paired runs. For a must-not-trigger case, add `min: 0`, `max: 0`, `arm: both`.

```markdown
---
type: tool_used
tool: Skill
input_match: '"skill"\s*:\s*"(?:[\w-]+:)?my-skill"'
---
```

File contents:

```markdown
---
type: regex
pattern: '^## \[Unreleased\]'
flags: m
target: { source: file, path: CHANGELOG.md }
---
```

Outcome rubric. Keep it short, with concrete conditions.

```markdown
---
type: llm
---

PASS if <observable condition>.
FAIL if <observable failure>.
```

## Run

```bash
# Pilot: one arm, one run
claude plugin eval <plugin-root> --tag my-skill --runs 1 --ablation none --allow-tools Write Edit --no-publish
# Paired: 3 runs with and 3 without the plugin — the only run that yields a Δ
claude plugin eval <plugin-root> --tag my-skill --allow-tools Write Edit --no-publish --max-cost-usd 10 --judge-model sonnet
```

- Every run is a real session on the user's account. Ask before running.
- `--ablation none` is for pilots only. It produces no `W/OUT` arm, so a suite only ever run that way has never been measured against the baseline.
- `--judge-model sonnet` for `llm` and `baseline` graders; the default judge is noisy. Deterministic graders ignore it.
- Bash, Write, Edit, and WebFetch need `--allow-tools`. The plugin's MCP servers don't start.
- The default `--threshold 1.0` exits 1 if any case is imperfect.
- Target the repo path. `name@marketplace` tests the installed cached copy instead.
- Pin `--model` when comparing runs, and test each model the skill will run on.
- `claude plugin eval init` interactively proposes cases for an existing plugin.

## User-invoked skills

For `disable-model-invocation: true` skills:
- Put `/plugin-name:skill-name args` in the prompt body; `-p` expands it.
- Use `--ablation none` and grade the output, because the no-plugin arm has no such command.
- The skill-fired indicator may not fire for slash expansion.
