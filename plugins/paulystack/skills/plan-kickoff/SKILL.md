---
name: plan-kickoff
description: Start a plan-mode research and implementation planning session. Use when the user wants deep investigation, latest-docs verification, parallel subagent exploration, and high-effort reasoning before writing any implementation. Invoke with the objective as the argument.
disable-model-invocation: true
effort: xhigh
argument-hint: '[objective — include language/version if relevant, e.g. "Refactor X in Zig 0.15.2"]'
---

# Plan-Mode Kickoff

**Objective:** $ARGUMENTS

Before proposing any implementation, ALWAYS execute this research pass in order:

1. **Investigate the codebase** — use Grep, Glob, Read to map every file relevant to the objective. Actively search for existing functions, utilities, and patterns that can be reused; do not propose new code when a suitable implementation already exists.
2. **Verify against latest docs** — the trigger is what the _implementation_ will call, not what the objective names. Objectives are usually codebase-relative ("delete rows from the documents table") and omit the stack; infer the stack from Step 1's findings instead: imports in the impacted files plus version pins in lockfiles (`pyproject.toml`/`uv.lock`, `package.json`/`package-lock.json`, `go.mod`, `Cargo.toml`, `Gemfile.lock`, etc.). For each language/SDK/framework/library the change will actually touch, consult current docs at the pinned version via Context7 MCP first, falling back to WebFetch/WebSearch. Stay narrowly scoped — skip docs for adjacent stack pieces the change won't touch (e.g. don't pull Postgres docs if you're only writing ORM calls; don't pull HTTP framework docs if you're only writing a Temporal workflow). Flag any API you cannot verify rather than guessing.
3. **Search the web when needed** — if the problem space has known prior art, community patterns, or recent breaking changes, search before deciding.
4. **Trigger parallel subagents to explore different paths and solutions** — dispatch Explore or Plan subagents in parallel; fan out across independent strands rather than one long sequential pass. All strands are subject to the **Strand barrier** below.
5. **Synthesize into an implementation plan** — only after every dispatched strand has returned and been verified, produce the plan. Do not start writing code until the plan is explicit. Remember you are in PLAN MODE.

## Strand barrier

Governs step 4, and applies to every parallel fan-out in the session — including the Explore/Plan agents dispatched by plan mode's own phases. Steps 1–3 are exempt: narrate codebase, docs, and web findings as you go.

**Dispatch** — name the roster first ("3 strands: A, B, C"), then launch all of them in a single message, asking each strand to cite the `file:line` or doc URL behind every claim. A strand inherits nothing from this session: not your findings, not the files you read, and for Explore and Plan not even CLAUDE.md — its prompt is the whole of what it will ever know. Write each one to stand alone — objective, the step 1–3 facts it cannot look up and where they came from, the output shape, the bounds of its lane. Hand it evidence, never the conclusion you expect. Never trickle-dispatch.

**Hold** — subagents return one at a time, and each return wakes you. A returning strand is not a turn. Until every strand on the roster is back, the only thing you may emit is a bare progress line ("2 of 3 back — holding"). Do not state findings, conclusions, recommendations, or risks; do not write or revise the plan file; do not ask the user clarifying questions; do not dispatch a replacement for a strand still outstanding.

**Verify** — once the roster is complete (or any lone subagent returns), check each claim the plan would change on if it were false, unless you've already seen its evidence yourself. Check first — these are where subagent findings have failed:

- absence claims ("no existing helper", "only caller", "not documented", "nothing comparable exists") — rerun the search yourself; absence inferred from a failed fetch or a keyword search is unverified, not negative
- claims that just confirm the premise you put in the strand's prompt
- uncited claims, and claims contradicting another strand or your own findings
- one anecdote generalized into a rule, or evidence from a different version, stage, or scope than asked

Cited codebase `file:line` findings hold up well; re-read them only when the plan hinges on them. A Plan agent's design is a proposal — check only the facts it rests on. Verify by reading the source yourself, not through another subagent: one handed a claim tends to confirm it. One refuted claim puts that strand's other load-bearing claims in scope and drops whatever it built on it. A claim you can't confirm stays out of the plan's premises unless marked unverified in the plan file. Until Release, emit only a bare progress line; then state in one chat line which claims you checked and what you corrected.

**Release** — once verified, synthesize once, reading all strands together. State agreement once. Resolve conflicts explicitly on evidence (`file:line`, doc citation), never by arrival order — the last strand back gets no extra weight. If a gap remains, dispatch a whole new round under this same barrier rather than a one-off follow-up.

Before writing out the solution, please let me know if you need anything to clarify. Ask before dispatching strands or after the synthesis — never between strand returns.

If you feel after your research that we do not need to create an implementation plan, then don't create a new plan and let me know that no plan is required and why it is no longer required.
