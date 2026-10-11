# Verification report

Contents: [Capture](#capture) · [Header](#header) · [Run block](#run-block) · [Excerpt](#excerpt) · [Rules](#rules) · [Investigation run](#investigation-run) · [PR review run](#pr-review-run) · [Example](#example)

One folder per branch, so the report and everything it quotes sit together:

```
<home>/.claude/verifications/<repo>/<branch-slug>/
├── report.md            the report; each run appends one block, earlier runs are never rewritten
├── cross-exam-r<n>.md   call-saul's answers for run <n>, one section per round
└── logs/r<n>-<n.k>.log  each step's full output
```

`<home>` comes from the skill's Context; the slug turns `/` into `-`, or is `detached-<sha7>`. The header is written when the folder is created.

Old layout (`<repo>/<branch-slug>.md` with logs in `<repo>/artifacts/<branch-slug>/`): before appending, move the report to `<branch-slug>/report.md` and the logs to `<branch-slug>/logs/`, then rewrite only the `artifacts/<branch-slug>/` prefix in its log paths to `logs/`. That path fix is the one edit earlier runs ever get.

## Capture

The harness hands back only a preview of long output (the first 2,000 characters past ~30k, or a head-and-tail excerpt for a failure), so the report quotes the log, never the preview.

- Run each proof command as `{ <cmd>; } > <log> 2>&1; echo "exit=$?" | tee -a <log>`, after `mkdir -p` on the `logs/` dir. The log's last line holds the exit code, so a reviewer can check `Exit:` against it. Read the log with `tail` or `grep` before writing the step.
- A server or worker you start writes its log to the same dir. MCP and browser steps save the response, snapshot, or screenshot there instead.
- `Ran:` shows `<cmd>` without the redirect, exactly as run: never shortened or paraphrased. Secrets become `<REDACTED:NAME>`. Prefix `cd <dir> &&` when it didn't run from the repo root.

## Header

```
# Verification: <branch> · <repo>
PR: #<n | none yet> · Profile: <path> (<fresh | drifted: files | no profile>)
```

## Run block

```
## Run <n> · <stage> · <sha7> · <YYYY-MM-DD HH:MM>
Task: "<the user's request, verbatim; trimmed with … only for length>" | follow-up: "<the follow-up, verbatim>"
Plan: <path to the approved plan the work comes from; omit the line when there is none>
Playbook: <name> · Done means: <the playbook's criteria for this task, one line each if several>
Report must show: <the playbook's "Report must show" line>

### <n>.<k> <what this step proves>
Why: <the done criterion it serves>
Expect: <the output that would prove it, written before running>
Ran:
    <exact command>
Exit: <code>
Output (<kept> of <total> lines · [full log](logs/r<n>-<n.k>.log)):
    <excerpt>
Verdict: seen | inferred (<from what>) | not checked (<why>) | failed  (an optional ` · <note>` explains only this step's output)

### How this completes the playbook
- <criterion>: steps <n.1>, <n.3> · seen
- <criterion>: not met · <what's missing> · <the command that would close it>
Changed files:
- `<path>`: <what changed> · justified by <n.k> | asked for by the user
Reverted: <edits tried and backed out, with why> | none
Not checked: <each gap the session could close but didn't, with the command that would> | none
Your checks:
- [ ] <what to do, e.g. "open /settings and click Save"> → <what you should see, e.g. "the toast says Saved with no error line">
Cross-exam: call-saul · <R> rounds · <N> objections · answered by <steps> | standing: <objection> | no objections · [cross-exam-r<n>.md](cross-exam-r<n>.md) | skipped (fast path) | not run (<why>)
Pending: <next stage, e.g. staging after deploy> · planned checks: <one line each>
Verdict: proven | partly proven | not proven
```

## Excerpt

- 20 lines or fewer: copy the output whole.
- Longer: keep the lines `Expect` names, error and summary lines (test totals, the `FAILED` or traceback line), and the last 3 lines. Mark each cut `… [<N> lines cut]`, and never cut inside a line that proves something.
- Prod and staging excerpts keep only the lines that prove the point, with customer data redacted.

## Rules

- Every criterion in the completion block cites a step. A check that has no step of its own, like the test suite, gets one.
- `Cross-exam: not run` caps the verdict at `partly proven` (an investigation: `partly answered`): nobody tested the evidence.
- Freshness: a step that proves the final code counts as `seen` only if it ran after the last code edit. Re-run an older one, or mark it `inferred (ran before edit to <file>)`. Steps meant to show the code before the change (a repro, a baseline, a pin) are exempt.
- Every file in `git diff --stat` appears under `Changed files`. A file no step or user request justifies gets reverted before the run is written, not listed.
- Staging and prod re-checks change no files; they write `Changed files: none`.
- Every step block stands alone and is written once. No summary after the completion block.
- A before/after pair uses the identical command.
- `seen` only for output you got in this run. Output reused from an earlier run is `inferred (run <n>)`.
- A verdict note explains only what this step's output shows. Earlier runs, history and intentions don't belong there; a claim worth making gets its own step.
- When the work comes from an approved plan, `Plan:` names it, and its Verification section is part of the requirements alongside `Task:`.
- `Your checks` holds what only the user can do, one action and its expected result per line. Carry forward every unticked item from the previous run; an item the user reports done becomes `- [x] … · you confirmed: "<their words>"` and counts as `inferred`, never `seen`.
- Inconclusive or wrong-surface proof is `not checked` or `failed`, never `seen`.

## Investigation run

Path: `<home>/.claude/verifications/<repo>/investigations/<YYYY-MM-DD>-<slug>/report.md`, with `cross-exam-r<n>.md` and `logs/` beside it, the same shape as a branch folder. A follow-up on the same question appends a run to the same file. Step blocks, Capture, and Excerpt are unchanged; the header and closing block differ:

```
# Investigation: <slug> · <repo>
Profile: <path> (<fresh | drifted: files | none>)

## Run <n> · investigation · <YYYY-MM-DD HH:MM>
Task: "<verbatim>" | follow-up: "<verbatim>"
Playbook: investigation · Done means: <the criteria for this question>
Report must show: <the playbook's line>
Window: <absolute start–end, tz> · Sources given: <thread, file, image, or none>

### <n>.<k> … (step blocks)

### Answer
<the answer, 1–4 lines>
- <claim>: steps <n.1>, <n.3> · seen
Ruled out: <other explanation> · step <n.k>
Sources checked: <each thread, dashboard, query target, file, image; "nothing relevant" where so>
Not checked: <gap the session could close but didn't, with the query that would> | none
Your checks:
- [ ] <the query or look only the user can do> → <what would confirm or change the answer>
Would change the answer: <evidence>
Cross-exam: <as in the run block>
Verdict: answered | partly answered | not answered
```

An investigation changes no files, so it has no `Changed files` or `Reverted`, and Freshness doesn't apply. Every other rule does.

Example step, from an MCP call:

```
### 1.3 Errors before the first timeout
Why: rules out the database as the cause
Expect: no `connection refused` lines for service:export before 02:14
Ran:
    search_datadog_logs(query="service:export env:prod \"connection refused\"", from="2026-10-03T22:00:00-03:00", to="2026-10-04T02:14:00-03:00")
Exit: ok
Output (1 of 1 lines · [full log](logs/r1-1.3.log)):
    {"data": [], "meta": {"page": {"after": null}}}
Verdict: seen · nothing found, so the database was reachable until the first timeout
```

## PR review run

Path: `<home>/.claude/verifications/<repo>/reviews/pr<N>/report.md`, with `cross-exam-r<n>.md` and `logs/` beside it. A re-review of the same PR appends a run. Step blocks, Capture, and Excerpt are unchanged; every step is read-only. The header and closing block differ:

```
# PR review: PR #<n> <title> · <repo>
Head reviewed: <sha7> (<matches PR head | behind by <k>>) · Base: origin/<base>

## Run <n> · pr-review · <YYYY-MM-DD HH:MM>
Task: "<the pr-recon arguments, verbatim>"
Playbook: pr-review · Done means: <the playbook's criteria>
Report must show: <the playbook's line>
Sources: PR #<n> + <related PRs> + <services read>

### <n>.<k> … (step blocks)

### Findings
- 1 · <label> · <title>: trigger <n.k>, path <n.k>, impact <n.k> · ruled out: <explanation> · <n.k>
- 2 · question · <title>: <what no read-only command could settle>
Dropped: <candidate> · <why> · <n.k>
Related read: <#N> <n.k>, …
Requirements: <requirement> · Met | Partial | Missed | Unclear · source <n.k> · diff <n.k>
Cross-exam: <as in the run block>
Verdict: proven | partly proven | not proven
```

A PR review changes no files, so it has no `Changed files`, `Reverted`, or `Pending`, and Freshness doesn't apply. `Your checks` appears only when a hop needs something only the user can see (a dashboard, a private repo). Every other rule applies.

## Example

```
# Verification: fix-calc-add · calc
PR: #none yet · Profile: ~/.claude/verify-profiles/calc/profile.md (fresh)

## Run 1 · local · 4e1a9c2 · 2026-09-27 14:05
Task: "`python3 calc.py add 2 3` prints 23 instead of 5. Fix it."
Playbook: bug · Done means: same repro fails then passes; root cause at file:line; neighbor op checked; existing tests pass
Report must show: the repro command, failing output, the root cause line, passing output from the same command, and any neighbor input you checked

### 1.1 See the bug
Why: bug needs the failure seen before editing
Expect: 23, not 5
Ran:
    python3 calc.py add 2 3
Exit: 0
Output (1 of 1 lines · [full log](logs/r1-1.1.log)):
    23
Verdict: seen

### 1.2 Root cause
Why: root cause named with the observation that confirms it
Expect: the args reach add() as strings
Ran:
    python3 -c "import sys; print(repr(sys.argv[2:]))" 2 3
Exit: 0
Output (1 of 1 lines · [full log](logs/r1-1.2.log)):
    ['2', '3']
Verdict: seen · calc.py:9 passes argv strings to add(), so + concatenates

### 1.3 Same repro after the fix
Why: same repro passes after
Expect: 5
Ran:
    python3 calc.py add 2 3
Exit: 0
Output (1 of 1 lines · [full log](logs/r1-1.3.log)):
    5
Verdict: seen

### 1.4 Neighbor op
Why: same class of input for another operator
Expect: 2
Ran:
    python3 calc.py sub 5 3
Exit: 0
Output (1 of 1 lines · [full log](logs/r1-1.4.log)):
    2
Verdict: seen

### 1.5 Existing tests
Why: existing tests still pass
Expect: OK with 0 failures
Ran:
    python3 -m unittest -v
Exit: 0
Output (5 of 41 lines · [full log](logs/r1-1.5.log)):
    test_add (test_calc.CalcTest.test_add) ... ok
    … [36 lines cut]
    ----------------------------------------------------------------------
    Ran 38 tests in 0.004s
    OK
Verdict: seen

### How this completes the playbook
- Same repro fails then passes: 1.1, 1.3 · seen
- Root cause: 1.2 · seen
- Neighbor input: 1.4 · seen
- Existing tests: 1.5 · seen
Changed files:
- `calc.py`: main() converts args to int before calling the op · justified by 1.2, 1.3
Reverted: a str() guard inside add(); 1.3 passed without it
Not checked: none
Your checks: none
Cross-exam: call-saul · 2 rounds · 1 objection · answered by 1.5 · [cross-exam-r1.md](cross-exam-r1.md)
Pending: none (no deploy pipeline in profile)
Verdict: proven
```
