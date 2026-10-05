# Figures

How to decide whether the trace or a bundle gets a figure, which kind, and how to draw it so it teaches instead of decorating. Page-level rules (tokens, safety, validation) are in [html-page.md](html-page.md).

## Contents
- [Does it get a figure?](#does-it-get-a-figure)
- [Change type → figure](#change-type--figure)
- [Visual grammar](#visual-grammar)
- [Layout without a layout engine](#layout-without-a-layout-engine)
- [Stepped animation (`deep` only)](#stepped-animation-deep-only)
- [Forbidden](#forbidden)

## Does it get a figure?

**The trace** gets one figure: the example input's path, with numbered hops. Skip it when the trace has ≤3 hops and the hop blocks alone make the path clear.

**Other bundles**, decided in this order:

1. **No figure** for renames, moves, extractions, formatting, type-only edits, dependency bumps, test-only changes, or a small one-line logic fix. Prose plus a short excerpt is enough.
2. **Stepped animation** only when the tier is `deep` AND the change alters *when or in what order* things happen AND the sequence has ≥3 steps.
3. **Static figure** when the change alters *structure* or *which branch runs*: who calls whom, where a responsibility lives, what a schema holds, which transitions exist.
4. **Deletion test** (applies to the trace figure too). Imagine the figure removed. If the prose alone would be as clear, remove it.

When a timing change could be shown either way, prefer a static strip of small panels (one per step) on `explain`, and the animation on `deep`, with the strip kept as its fallback.

## Change type → figure

| Change | Figure | What to highlight |
|---|---|---|
| Trace (any change on a runtime path) | The example input's path left→right through every module it passes, changed or not; numbered hops | Changed hops in diff tokens, unchanged hops `--ctx`; labels carry the example's real values |
| Cross-service call | Sequence lanes, one per service, each lane framed and labeled with its service or repo name | Each crossing labeled with transport + route or topic + payload (`HTTP POST /v1/charges {order_id, amount}`); sync = solid line, filled arrowhead; async (queue, event) = dashed line, open arrowhead; a key for both |
| Responsibility moved (refactor) | Before/after pair of the same board, nodes in identical positions | The moved responsibility line and the rewired edges; everything else `--ctx` |
| Cache added or changed | Read path with hit and miss branches; the cache node shows `key → value, TTL` | Hit vs miss path; invalidation edge dashed |
| Retry, timeout, idempotency | Sequence lanes, time top→bottom (animated on `deep`) | The failure point and the new retry/backoff messages with their delays |
| Concurrency, locking, ordering | Two lanes on a shared timeline (animated on `deep`) | The race window, then how the fix closes it |
| State machine | Static state chart; transitions labeled `event [guard] / action` | New or changed transitions; removed states shown dashed |
| Schema or data migration | Before/after field lists + a deploy-step timeline (write both → backfill → switch reads → drop) | Added/removed fields; the rollback point |
| UI state flow | Panel strip: event → state → rendered view | The new event or derived state and what re-renders |
| Algorithm swap | The same toy input run through old and new, side by side | The first step where they diverge |

## Visual grammar

1. **One question per figure.** The `<figcaption>` states the question the figure answers ("How does a failed payment release the seat hold?"). One diagram type and one abstraction level per figure; runtime flow and static structure never share a figure.
2. **Every edge is one-way and labeled with what travels on it**: a call with its arguments (`reserve(orderId, userId)`), an endpoint with its payload (`POST /retry → 202`), or a short verb phrase ("releases hold after 3 failures"). Never a bare arrow, "uses", or "calls". The label reads correctly out loud in the arrow's direction.
3. **Numbered hops match the hop blocks.** Each numbered badge in the figure has exactly one hop block below with the same number, and vice versa.
4. **Small nodes, few nodes.** At most ~9 nodes; group consecutive unchanged hops into one node (`auth + rate-limit middleware`) or split into two figures beyond that. Node text is at most 3 lines: name, (technology or file), and 1–3 responsibilities or fields, ending in `…` when cut.
5. **Nothing floats, nothing extra.** Every node is connected. Every node is either touched by the diff or needed to explain it. Every node is named in the surrounding prose.
6. **Figures that build on each other share a frame.** Keep the same canvas size and node positions so only the change moves. When a later figure supersedes an earlier label, edit the label; don't keep the old one alongside.
7. **One encoding for change, page-wide.** Changed elements use `--add`/`--del`/`--mod` with their second cue; context uses `--ctx`. No other color carries meaning. Add a legend only when an encoding appears in more than one figure, and define it once.
8. **Concrete values.** Labels use the example input (ids, TTLs, retry counts, status codes), not placeholders like `data` or `value`.

## Layout without a layout engine

Hand-placed coordinates are where generated diagrams fail: overlapping boxes, text spilling out, arrows ending in empty space. Avoid that structurally:

- **Declare a grid before drawing.** Name the columns by role (e.g. caller | handler | service | store) and the rows by lane or step. Compute every node's position from its cell (`x = pad + col × colWidth`, `y = pad + row × rowHeight`). No free-hand coordinates.
- **Size boxes from their text.** Render the `<text>` first, measure with `getBBox()`, then size the `<rect>` with padding. Hardcoded widths overflow because system fonts differ per OS.
- **Orthogonal edges.** Paths leave from the midpoint of a box side and turn at right angles (`M x1 y1 H xm V y2 H x2`). When several edges leave one side, use the 1/3 and 2/3 points. In sequence lanes, each message is a horizontal line on its own row, so messages can't collide.
- **Edge labels** sit at the midpoint of the edge's longest segment, offset a few px away from nodes, with a background-colored pad behind the text. 1–4 words, or one short signature.
- **Mark the structure:** each node has an `id`; each edge has `data-from` and `data-to` naming node ids. The validation loop uses these to catch arrows to nowhere.
- **Responsive.** `viewBox` on every `<svg>` with `width: 100%; height: auto`. Below ~520px, switch to a second, vertical coordinate map (columns become rows) rather than shrinking text below 11px.

## Stepped animation (`deep` only)

**Behavior**
- **Starts paused** on step 0: the full cast of parts, labeled, nothing moving. Nothing animates on page load.
- **Reader-paced controls:** Prev, Next, Play/Pause, a scrub slider with one tick per step, and ←/→ keys when the figure has focus. A visible "Step k / N" counter.
- **One caption per step**, one sentence, next to the figure, updated at the same moment as the frame.
- **One kind of change per step.** Transitions ≤~1s. At most ~12 steps; split a longer sequence into two figures.
- **Highlight only what changes** in that step; dim the rest to `--ctx`. When a change propagates along a path (a request, a retry), the highlight travels along that path.
- **One example input throughout**, the same one as the big picture and the trace.
- **Static at both ends.** The first and last frames each work as a readable before/after figure.
- **"Show all steps"** toggle renders every step as a strip of small panels with their captions.
- **Reduced motion and no-JS fallback.** The small-panel strip is in the HTML by default. Script hides it and shows the animated figure only when `prefers-reduced-motion: reduce` does not match.

**Mechanics** (write fresh for each page; keep it small)
- A `steps` array of `{ caption, highlight: [ids], edge: id }`.
- `render(i)` removes the active class everywhere, adds it to the step's ids and edge, sets the caption and counter, and syncs the slider.
- To move a packet along an edge, animate with the Web Animations API (`element.animate`) and position the packet with `path.getPointAtLength(t × length)`. Seek by setting the animation's `currentTime`; pause and play through the `Animation` object.
- Don't use SMIL (`<animate>`, `<animateMotion>`): it can only pause the whole `<svg>`, not one step.

## Forbidden

- Animating anything that doesn't change over time; autoplay; looping or idle motion; entrance, parallax, or scroll-triggered effects.
- Content that only appears on hover.
- Bare box-and-arrow diagrams, unlabeled edges, a figure without a question caption.
- Icons, logos, or decorative illustrations.
- LOC or file-count charts, "key takeaways" boxes.
- Mermaid or any library or CDN.
- Whole hunks or side-by-side full files inside figures.
- Quizzes or "what do you think happens next?" prompts.
