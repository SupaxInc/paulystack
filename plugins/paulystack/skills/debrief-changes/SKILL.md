---
name: debrief-changes
description: Debriefs the uncommitted git changes in the current repo (staged, unstaged, untracked) as a scroll-once review guide grouped by concept, opening with a short terms list. For complex diffs it also writes a local HTML page tracing an example request through the code, across services if needed. Use when the user wants to understand what changed before reviewing it, such as "debrief my changes", "what did you just change?", "walk me through this diff", or "explain what the agent edited".
argument-hint: "[brief|explain|deep] [output.html]"
allowed-tools: Bash(git status *) Bash(git diff *) Bash(git ls-files *) Bash(echo *) Bash(mkdir -p *) Bash(open *) Bash(xdg-open *) Bash(wc *) Read Grep Glob Write
---

# Debrief changes

## Current changes

<!-- Each command ends in an echo fallback: a failing injected command aborts the whole skill, so Claude would never see these instructions. -->

Working tree:
!`git status --short --branch -uall 2>&1 || echo "GIT UNAVAILABLE"`

Untracked files (no diff below; Read them or `wc -l` them for content and LOC):
!`git ls-files --others --exclude-standard 2>/dev/null || echo "GIT UNAVAILABLE"`

Stat:
!`git diff HEAD --stat=200 2>/dev/null || git diff --cached --stat=200 2>/dev/null || echo "GIT UNAVAILABLE"`

Full diff (if this arrives as a file path plus preview, run `git diff HEAD -- <path>` per bundle instead):
!`git diff HEAD 2>/dev/null || git diff --cached 2>/dev/null || echo "GIT UNAVAILABLE"`

## When this doesn't apply

- Working tree says "not a git repository": say so in one line and stop.
- A section says `GIT UNAVAILABLE` for another reason: gather the same status, untracked list, and diff yourself, then continue with the steps below. Don't describe the workaround; the final message still opens with the TL;DR heading.
- No uncommitted changes: say so in one line and stop.
- Committed history, a PR, or a branch comparison; the user asks how existing code works rather than what changed; bug or security review. Use your normal approach for those. Reading unchanged code to explain the diff is in scope.

## 1. Pick the tier

By default, assess the diff and pick the tier yourself.

An override wins only when present:
- An `ARGUMENTS:` line: `brief`, `explain`, or `deep`, and/or a path ending in `.html` to write the page to.
- The user's own words: "quick text summary" / "no page" → `brief`; "explainer page" → `explain`; "in depth" / "animated" / "interactive" → `deep`; a named `.html` path.

Otherwise the highest tier whose signals fire wins:

| Tier | Signals | Output |
|---|---|---|
| `brief` | Only renames, formatting, docs, lockfiles, dependency bumps, or value-only config; Scattered mode; or ≤~80 non-test changed lines with no new symbol and no control-flow change | Terminal debrief only |
| `explain` | A new exported type or module with logic; a new schema, migration, endpoint, or enum variant; a signature or error-contract change whose callers live in other files (Grep them); real "A writes X, B reads X" glue across directories; a call added or changed across a service boundary (HTTP, gRPC, queue, event); ≥2 real bundles with at least one med/high risk | Terminal debrief + HTML page with static figures |
| `deep` | `explain` signals plus a change to *ordering or timing*: retry/backoff, async or lock ordering, races, cache hit/miss/eviction, queue/backpressure, recursion | Terminal debrief + HTML page that may add stepped animations for those changes |

Cap at `explain` (organized by theme) when the diff touches 15+ files or ~2,000+ changed lines, unless the user or an argument asked for `deep`.

## 2. HTML page (`explain` and `deep` only)

Skip this section entirely for `brief`.

1. Read [references/html-page.md](references/html-page.md) and [references/figures.md](references/figures.md). When the traced flow calls another service (HTTP, gRPC, queue, event), also read [references/tracing.md](references/tracing.md).
2. Decide the page from this diff: the example input and the path it takes, which bundles get a figure, what kind, and which get none. There is no template; the references give rules, not a layout.
3. Write the page to the override path if given, else `~/.cache/debrief-changes/YYYY-MM-DD-<repo-dir-name>-<slug>.html` (`mkdir -p ~/.cache/debrief-changes` first).
4. Run the grep validation loop from html-page.md. Fix and re-check, at most 2 rounds.
5. Open it with `open <path>` (macOS) or `xdg-open <path>` (Linux). If opening fails, carry on.

Write the page before printing the debrief, so the debrief is the final message. Status about the page (open failed, a validation check still failing) goes in parentheses on the `Explainer:` line, e.g. `Explainer: <path> (couldn't open automatically)` — never as a sentence before the TL;DR heading.

## 3. Terminal debrief (always)

A scroll-once review guide: each bundle is one self-contained block, so the reader never flips back to finish reviewing a group.

### Hard rules

1. **Row-major, scroll-once.** Each bundle: heading → 1-4 line rationale → optional `Why bundle:` line → **blank line** → `Files` label + numbered file list → **blank line** → inline ⚠ watch lines. The two blank lines are mandatory — they separate the file list from the prose.
2. **Each bundle is written exactly once. Stop after the last bundle (or `## Skim`).** No re-listing, no recap, no "to summarize", no closing "worth knowing" or "overall" paragraph. A point worth making belongs as a rationale line or a `⚠` line in its bundle. If you're about to emit a heading you've already written, you're done — stop.
3. **No trailing sections.** No `## Watch-outs`, `## Cross-cutting`, `## Dependency order`, or `## Summary`. All per-bundle info stays inline.
4. **Rationale: 1-4 lines, one idea per line, no padding.** Each line carries one of: action (what changed), mechanism shorthand (only when non-obvious), motivation (only when surprising, e.g. "hand-rolled because std.http.Client collapses error.WouldBlock"), invariants preserved (e.g. "tmux byte-identical"), or cross-bundle links. A line that restates the heading or the file list is dropped.
5. **Cross-bundle references in free form:** "called by bundle 5", "consumes `metadata.tty` from bundle 6", "extends the pattern in `registry.zig`".
6. **Concept-first, not layer-first.** A feature spanning API + types + UI + tests is ONE bundle.
7. **Real glue vs fake glue.** A bundle needs a dependency or shared intent ("A writes state B reads", "B consumes A's exported type"). "Both config", "both in `src/`" is fake glue — use Scattered mode.
8. **No filler concept names:** `Misc`, `Various`, `Cleanup` (unless a real cleanup pass), `Improvements`, `Updates`, bare `Refactor`.
9. **Numbered file list under a plain `Files` label** (never a `##`/`###` heading). One line per file: `  N. \`path/from/repo/root\` (±LOC)   <one-liner>`, numbering restarting at 1 per bundle, `(NEW, NN)` for new files, gaps padded so LOC and descriptions roughly align. Don't restate what `git diff --stat` already shows.
10. **Reading order in the heading:** `## N. <Concept> · risk: <low|med|high> · <read first | read after K | independent>`. Number bundles so dependencies read forward — never `## 1. … · read after 3`.
11. **Risk tags only where they add signal.** Skip `· risk: low` on trivial bundles.
12. **Watch lines are `file:line` specific, prefixed `⚠`, one line each.** Never generic ("be careful with auth"). No risk, no `⚠` line.
13. **Skim bucket for boring files:** mirror tests, docs, formatting, barrel additions go in the final `## Skim`, never their own bundle.
14. **Open with the TL;DR heading:** `## TL;DR · <tier> — <one-phrase reason for the tier>`, then one sentence on what the change does as a whole. For `explain`/`deep`, the next line is `Explainer: <path>`. Then the `Terms` block (rule 16), then bundle 1. The heading is the first line of your final message. Nothing before it: no acknowledgement ("No page — here's the text debrief", "Here's the debrief"), no note about how you gathered the diff, no validation or progress report.
15. **`brief` only:** one ASCII flow diagram (≤10 lines, inside the bundle it explains) is allowed when a flow has ≥3 hops the `Why bundle:` line can't carry.
16. **Terms block before bundle 1.** The reader is often new to the codebase or stack.
    - Always on `explain`/`deep`; on `brief` only when the debrief uses a term from the list below.
    - A plain `Terms` label (no heading, no numbers), then ≤5 lines of `  <term> — <definition>`.
    - Terms worth defining: codebase-specific names (services, modules, domain words), framework or stack concepts, named patterns (idempotency key, backoff). Skip general programming words (function, HTTP, JSON).
    - Swap a term for plain words when you can; define only what's left. Definitions are ≤~12 words and use this change's concrete values: `backoff — wait longer after each failed try (1s, 2s, 4s)`.
    - Keep one name per concept everywhere after. No other non-obvious term appears undefined.

### Bundle shapes

**Concept bundle (2+ files):**
```
## N. <Concept name> · risk: <low|med|high> · <read first | read after K>
<1-4 lines of rationale>
Why bundle: <real-glue sentence>

Files
  1. `path/to/file.ts` (±NN)     <one-liner>
  2. `path/to/other.ts` (±NN)    <one-liner>

⚠ <file:line> <specific concern>
```

**Independent file (single file)** — same, but `· independent` in the heading and no `Why bundle:` line.

**Skim (always last, never numbered):** no risk tag, rationale, or watches.
```
## Skim
  1. `path/one.ts` (±NN)         <very short one-liner>
```
With 4+ skimmables, a comma-separated line is fine: `README.md, .gitignore, tests/foo.test.ts (mirror)`.

### Worked example

A change adding OAuth login (3 files) + extracting a Session type (1 file) + a README tweak:

```
## TL;DR · explain — callback writes the session the login endpoint reads
Adds OAuth login + extracts Session type.
Explainer: ~/.cache/debrief-changes/2026-09-13-shop-oauth-login.html

Terms
  OAuth provider — the outside login service (Google) that vouches for the user
  callback — our URL the provider redirects to after login, carrying a one-time code
  refresh token — long-lived token that gets a new 1-hour access token

## 1. OAuth flow · risk: med · read first
Adds the OAuth provider, callback wiring, and login endpoint.
Flow: provider opens external auth → callback writes session → endpoint reads it on next request.
Why bundle: callback writes the session before the endpoint reads it — must be reviewed as one contract.

Files
  1. `src/auth/oauth.ts` (NEW, 87)     new provider; redirect + token exchange
  2. `src/auth/session.ts` (+12)       callback writes refreshed session
  3. `src/api/login.ts` (+24)          new endpoint reading session for redirect

⚠ oauth.ts:84 silently swallows refresh-token errors

## 2. Session type · risk: low · independent
Extracted from auth/session.ts so non-auth modules can import the type without pulling the runtime.
Consumed by bundle 1's session.ts; future caller is the user-profile module.

Files
  1. `src/lib/session.ts` (NEW, 8)     type-only re-export

## Skim
  1. `README.md` (+2)                  login flow note
```

Match this depth. Output ends after the last `## Skim` line.

### Calibration

- **Tiny (1–3 files):** no bundle numbering. TL;DR heading + sentence → blank line → optional `Terms` block → `Files` label + numbered list → optional `⚠` lines. No `## Skim`.
- **Medium (4–12 files):** full structure, 3–5 bundles + optional `## Skim`.
- **Huge (15+ files):** rationale 2 lines max, ~5 bundles + Skim. End the last bundle with exactly one line: `Want a deep-dive on bundle N?`
- **Scattered (no honest concept):** no bundles.
  ```
  ## TL;DR · brief — no shared concept
  Scattered tweaks — <one phrase, e.g. "5 small unrelated fixes">.

  Terms
    <term> — <definition>

  Files
    1. `path/one.ts` (±NN)    <one-liner> · risk: low
    2. `path/two.ts` (±NN)    <one-liner> · risk: med

  ⚠ <file:line> <specific concern>
  ```
  Use it when you can't write a non-filler concept name, or the only `Why bundle:` you can write is fake glue. Drop the `Terms` block when rule 16 doesn't call for it.
