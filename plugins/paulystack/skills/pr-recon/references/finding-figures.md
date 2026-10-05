# Finding figures

A review figure proves one claim: this input, through this path, reaches this line and breaks that. Tokens and page rules are in [review-page.md](review-page.md).

## Contents
- [Does it get a figure?](#does-it-get-a-figure)
- [Finding kind → figure](#finding-kind--figure)
- [Visual grammar](#visual-grammar)
- [Layout without a layout engine](#layout-without-a-layout-engine)
- [Stepped animation](#stepped-animation)
- [Forbidden](#forbidden)

## Does it get a figure?

1. **No figure** for a finding that lives on one or two lines (a wrong operator, an off-by-one, a nit, a naming question). The base vs head excerpt is the evidence.
2. **A figure** when the claim needs a path or a comparison to believe: a value travels across files, a consumer elsewhere breaks, an ordering goes wrong, or a deploy step leaves a gap.
3. **Deletion test.** Imagine the figure removed. If the hop blocks and excerpt would convince the reader just as well, remove it.

## Finding kind → figure

| Finding kind | Figure | What to highlight |
|---|---|---|
| Bad value reaches a line (null, unvalidated input, wrong unit, wrong id) | Path left→right: **trigger** (the concrete input that causes it) → hops → problem line → **impact** | Problem hop `--issue`, the damage node `--impact`, PR-changed hops `--head`, others `--ctx`. Edge labels carry the trigger's values (`id = undefined`) |
| Guard, check, or validation removed or weakened | Base vs head pair of the same small path, nodes in identical positions | The guard in the base figure `--base` (dashed, struck); in the head figure the edge that now skips it `--issue` |
| Contract change (signature, payload, response, schema, event) | Producer on the left fanning out to every consumer found, one row per consumer | The field that differs on each edge (`amount: int → string`). Each consumer is marked `updated` (`--head`), `breaks` (`--issue`), or `not read — <reason>` (dashed boundary, per [cross-service.md](cross-service.md)) |
| Race, ordering, retry, or idempotency | Two lanes (one per actor, request, or worker) on a shared timeline running top→bottom, one message per row | The interleaving that breaks, with the window between the two steps shaded `--issue` and the resulting bad state `--impact` |
| Deploy or migration ordering | Timeline of deploy steps (migrate → deploy service A → deploy service B …) with which code version is live at each step | The window where old code meets the new schema or contract, `--issue`, and the rollback point |
| Config, flag, or env var | Grid: environments (rows) × where the value is set vs where it's read (columns) | The environment where it's missing or different, `--issue` |
| Changed behavior with no test | HTML table, not SVG: behavior cases (rows) × covered by which test | The uncovered case that the finding is about, `--issue` |
| Performance (N+1, unbounded loop, missing pagination) | The loop with a multiplier on its edge (`1 query per order × 500 orders`) | The multiplied edge `--issue` and the total cost at realistic size `--impact` |

When one finding fits two kinds, pick the one that answers the reader's first doubt ("can this input really get here?" → the path).

## Visual grammar

1. **Caption = the claim.** The `<figcaption>` states what the figure proves ("A logged-out `GET /profile` reaches `findOne` with no id"). One claim, one diagram type, and one level of detail per figure.
2. **Trigger to impact.** Path figures start at a concrete trigger a reader could reproduce and end at a visible effect. Never start mid-path, and never end at the problem line without its impact.
3. **Every edge is one-way and labeled with what travels:** a call with arguments (`getUser(undefined)`), an endpoint with its payload (`POST /charges {amount: "12.50"}`), or a short verb phrase. Never a bare arrow, "uses", or "calls".
4. **Numbered hops match the hop blocks.** Each badge in the figure has exactly one hop block with the same number, and the reverse.
5. **Few nodes.** At most ~7. Merge consecutive unchanged hops into one node (`auth + rate-limit middleware`). Node text is ≤3 lines: name, file or service, and one fact that matters to the claim.
6. **Nothing floats, nothing extra.** Every node is connected and named in the hop blocks. A node is there only if the claim needs it.
7. **One encoding, page-wide.** Only the five tokens carry meaning. Add a legend once, near the first figure that uses more than two of them.
8. **Concrete values** from the trigger scenario, never placeholders like `data` or `value`.

## Layout without a layout engine

Hand-placed coordinates are where generated diagrams fail: overlapping boxes, text spilling out, arrows ending in empty space.

- **Declare a grid first.** Name the columns by role (trigger | handler | service | store | impact) and the rows by lane or step. Compute every position from its cell (`x = pad + col × colWidth`, `y = pad + row × rowHeight`).
- **Size boxes from text.** Render each `<text>`, measure it with `getBBox()`, then size its `<rect>` with padding. Fixed widths overflow because fonts differ per OS.
- **Orthogonal edges** leave from the midpoint of a box side and turn at right angles (`M x1 y1 H xm V y2 H x2`). Several edges from one side use the 1/3 and 2/3 points. In lane figures, each message gets its own row.
- **Edge labels** sit at the midpoint of the edge's longest segment, with a background-colored pad behind the text. 1–4 words, or one short signature.
- **Mark structure:** every node has an `id` and every edge has `data-from` / `data-to` naming node ids (validation check 6).
- **Responsive:** every `<svg>` gets a `viewBox`, `width: 100%`, and `height: auto`. Below ~520px, switch to a vertical coordinate map (columns become rows) instead of shrinking text below 11px.

## Stepped animation

Only for race, ordering, or retry findings whose bad interleaving takes ≥4 steps. Everything else is static.

- Starts paused on step 0, the full cast labeled. Nothing moves on load.
- Controls: Prev, Next, Play/Pause, a scrub slider with one tick per step, ←/→ keys when the figure has focus, and a "Step k / N" counter.
- One one-sentence caption per step, updated with the frame. One kind of change per step, transitions ≤~1s, ≤10 steps.
- The last frame shows the bad state as a readable static figure.
- A small-panel strip of every step is in the HTML by default. Script swaps in the animated figure only when `prefers-reduced-motion: reduce` does not match.
- Mechanics: a `steps` array of `{caption, highlight: [ids]}` and a `render(i)` that moves the active class. Use the Web Animations API (`element.animate`), not SMIL.

## Forbidden

- Figures for one-line findings, or figures that restate the excerpt.
- Autoplay, looping, idle, hover-only, entrance, or scroll-triggered effects.
- Bare box-and-arrow diagrams, unlabeled edges, or a figure without a claim caption.
- Icons, logos, decorative illustrations, severity charts, and LOC charts.
- Mermaid or any library or CDN.
- Whole hunks or full files inside figures.
