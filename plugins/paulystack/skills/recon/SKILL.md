---
name: recon
description: Run a thorough investigation pass on an objective and return an inline analysis. Use when the user wants codebase mapping, latest-docs verification, web search, and parallel subagent exploration but NOT an implementation plan or code. Invoke with the objective as the argument.
effort: xhigh
argument-hint: '[objective — include language/version if relevant, e.g. "How does our auth handle refresh tokens in Next.js 15?"]'
---

# Deep Research

**Objective:** $ARGUMENTS

Execute this research pass in order. The deliverable is an **inline analysis of findings**, not an implementation plan and not code.

1. **Investigate the codebase** — use Grep, Glob, Read to map every file relevant to the objective. Trace existing functions, utilities, and patterns; cite each finding with `file:line` so the user can jump to it. Surface reusable prior art rather than reinventing it.
2. **Verify against latest docs** — the trigger is what the _analysis_ will reason about, not what the objective names. Objectives are usually codebase-relative ("how does our auth handle refresh tokens?") and omit the stack; infer the stack from Step 1's findings instead: imports in the relevant files plus version pins in lockfiles (`pyproject.toml`/`uv.lock`, `package.json`/`package-lock.json`, `go.mod`, `Cargo.toml`, `Gemfile.lock`, etc.). For each language/SDK/framework/library the analysis depends on, consult current docs at the pinned version via Context7 MCP first, falling back to WebFetch/WebSearch. Stay narrowly scoped — skip docs for adjacent stack pieces the analysis doesn't touch. Flag any behavior you cannot verify rather than guessing.
3. **Search the web when needed** — if the problem space has known prior art, community patterns, or recent breaking changes, search before concluding.
4. **Trigger parallel subagents to explore different paths** — dispatch Explore subagents in parallel; fan out across independent strands (current implementation, adjacent systems, alternative approaches) rather than one long sequential pass. All strands are subject to the **Strand barrier** below.
5. **Synthesize an inline analysis** — only after every dispatched strand has returned and been verified, report the findings directly in chat. Do NOT produce an implementation plan, and do NOT write code. Let the objective drive the structure — typical shape: what exists today, how it works, gaps or risks, open questions — but adapt freely. Cite evidence (`file:line`, doc URLs) for each substantive claim, and call out unknowns you could not resolve.

## Strand barrier

Governs step 4, and applies to every parallel fan-out in the session. Steps 1–3 are exempt: narrate codebase, docs, and web findings as you go.

**Dispatch** — name the roster first ("3 strands: A, B, C"), then launch all of them in a single message, asking each strand to cite the `file:line` or doc URL behind every claim. A strand inherits nothing from this session: not your findings, not the files you read, and for Explore and Plan not even CLAUDE.md — its prompt is the whole of what it will ever know. Write each one to stand alone — objective, the step 1–3 facts it cannot look up and where they came from, the output shape, the bounds of its lane. Hand it evidence, never the conclusion you expect. Never trickle-dispatch.

**Hold** — subagents return one at a time, and each return wakes you. A returning strand is not a turn. Until every strand on the roster is back, the only thing you may emit is a bare progress line ("2 of 3 back — holding"). Do not state findings, conclusions, recommendations, or risks; do not write out any part of the analysis; do not ask the user clarifying questions; do not dispatch a replacement for a strand still outstanding.

**Verify** — once the roster is complete (or any lone subagent returns), check each claim the analysis would change on if it were false, unless you've already seen its evidence yourself. Check first — these are where subagent findings have failed:

- absence claims ("no existing helper", "only caller", "not documented", "nothing comparable exists") — rerun the search yourself; absence inferred from a failed fetch or a keyword search is unverified, not negative
- claims that just confirm the premise you put in the strand's prompt
- uncited claims, and claims contradicting another strand or your own findings
- one anecdote generalized into a rule, or evidence from a different version, stage, or scope than asked

Cited codebase `file:line` findings hold up well; re-read them only when the analysis hinges on them. A strand's recommended approach is a proposal — check only the facts it rests on. Verify by reading the source yourself, not through another subagent: one handed a claim tends to confirm it. One refuted claim puts that strand's other load-bearing claims in scope and drops whatever it built on it. A claim you can't confirm goes under the analysis's unknowns, never stated as fact. Until Release, emit only a bare progress line; then open the analysis with one line on which claims you checked and what you corrected.

**Release** — once verified, synthesize once, reading all strands together. State agreement once. Resolve conflicts explicitly on evidence (`file:line`, doc citation), never by arrival order — the last strand back gets no extra weight. If a gap remains, dispatch a whole new round under this same barrier rather than a one-off follow-up.

Before delivering the analysis, ask the user if anything needs clarification. Ask before dispatching strands or after the synthesis — never between strand returns.
