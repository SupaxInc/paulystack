---
name: skill-authoring
description: Guides writing a new agent skill correctly and reviews existing SKILL.md files against skill-authoring best practices — specific trigger descriptions, no no-op instructions, the right degree of freedom, compact structure, and evals that beat a no-skill baseline. Use when the user wants to create or write a skill, turn a workflow or team conventions into a skill, or review, audit, or improve an existing skill.
argument-hint: '[what the new skill should do, or path to a SKILL.md to review]'
---

# Skill Authoring

Two modes, both judged by the checklist below:

- **Review**: the user points at an existing SKILL.md or asks to review, audit, or improve a skill.
- **Create**: the user wants a new skill.

## Checklist

**Right tool**
- The workflow is identical every run → a script in `scripts/`; a skill at most wraps it.
- The model already does the task well unprompted → no skill. The eval baseline decides, not intuition.

**Frontmatter**
- `name`: lowercase letters, digits, hyphens; ≤64 characters; no "anthropic" or "claude".
- `description`: third person ("Generates…", never "I can…" or "You can…"); what it does, then when to use it; key use case first; concrete triggers (file types, tools, phrases) so it matches on the first scan. It shares a 1,536-character budget with `when_to_use`, above which the listing truncates — so put the key use case first.
- `when_to_use`: extra trigger phrases and example requests, appended to `description` for matching. Same budget; use it only when the description is already carrying its weight.
- Specific beats pushy: an over-broad description ("web development") over-triggers. Enforce boundaries with near-miss evals rather than "do not use" clauses.
- `paths`: glob list that gates *automatic* activation to matching files. The better fix for a per-language or per-directory skill than widening the description — the skill still lists under `/` and still runs when invoked by name.
- `disable-model-invocation: true` only for side-effecting or manual-only workflows. Such skills can't be tested for triggering, only for output. Its mirror is `user-invocable: false`, for background knowledge the user shouldn't invoke.
- `context: fork` runs the skill in a subagent that cannot see the conversation — the body becomes its entire prompt, so a forked skill needs explicit instructions, not guidelines. Pairs with `agent` (`Explore`, `Plan`, `general-purpose`, or a custom one) and `background: false` to wait for the result in the invoking turn.
- `disallowed-tools` removes tools for the turn, for autonomous skills that must never call one.

**Content**
- Keep only what the model cannot infer: team conventions, data sources, exact formats or constraints a checker verifies, real gotchas.
- No-op test, sentence by sentence: would the model behave differently without it? If not, delete the whole sentence ("write clean code", "be thorough", explaining what a changelog is). A word too weak to beat the default is also a no-op; make it concrete or cut it.
- One concrete input/output example beats a paragraph of explanation.
- A skill that dispatches subagents says what goes into the delegation prompt: a subagent inherits nothing from the session, and `Explore`/`Plan` skip CLAUDE.md, so a session-acquired fact reaches it only by being written into that prompt.
- No dates or "until version X" notes; one term per concept.

**Degrees of freedom**
- Many valid approaches → goals and constraints, not step lists.
- Fragile or order-dependent → numbered steps with a validate → fix → repeat loop.
- Deterministic → the exact command, or a bundled script run via `${CLAUDE_SKILL_DIR}/scripts/…`, with each default commented with its reason.

**Harm guards** (skills make agents worse when heavy or brittle)
- Mark optional steps and give a fast path for simple cases.
- State when the skill does not apply, so the model keeps its native approach there.
- Give fragile steps a check that catches wrong output, plus a fallback.

**Structure**
- SKILL.md well under 500 lines. Move detail into reference files linked directly from SKILL.md (one level deep); a reference over 100 lines starts with a table of contents.

**Evals**
- ≥3 realistic should-trigger prompts (varied, including casual phrasing) and ≥2 near-miss prompts that must not trigger. Grow the set from real failures.
- Deterministic graders first (regex, file exists, tool used), then one rubric grader for the outcome. Grade the outcome, not whether the skill loaded on turn one.
- 3 runs per case, with and without the skill. Ship only when Δ > 0.
- Never copy eval answers into the skill.
- Case format and commands: [references/plugin-eval-cases.md](references/plugin-eval-cases.md).

## Create

1. **Gather facts from the user; never invent them.** Ask only what the request hasn't already answered:
   - prompts that should and shouldn't trigger it
   - what Claude gets wrong today without the skill
   - the facts it can't infer
   - where the skill lives

   Invented gotchas are the main way agent-written skills end up worse than no skill.
2. **Propose eval cases before drafting.** Offer to write them to the plugin's `evals/<skill>/`; write only with the user's OK.
3. **Draft the minimal SKILL.md** against the checklist and show it in chat. Prefer cutting to adding. Write the file only after the user approves the draft.
4. **Offer the eval run** (`claude plugin eval <plugin-root> --tag <skill>`), noting it spends usage; run it only on approval. Reading results:
   - Δ ≤ 0 → the skill isn't helping; rework it or don't ship it.
   - Δ ≈ 0 and the skill-fired grader failing → the description isn't matching; fix that first.
   - A grader that passes in both arms measures nothing; replace it.
   - Iterate with `--runs 1 --ablation none`; confirm at the default 3 runs before trusting a change.

## Review

1. Read the SKILL.md and every file it links.
2. Report findings by checklist section, most severe first. Each finding gives `file:line`, the problem, and the concrete fix, quoting any no-op sentences to delete. Include the line count and the description's character count.
3. Evals: if there are none, or they're weak, propose cases. If an existing suite shows Δ ≈ 0 on current models, recommend retiring the skill and keeping the suite as a regression check.
4. Report only. Edit the skill only when the user asks.
