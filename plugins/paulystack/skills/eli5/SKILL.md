---
name: eli5
description: Explain a topic like I'm a 5 year old. Use when the user types /paulystack:eli5 <topic> or asks for a dead-simple picture explainer of how something works. Writes a local HTML page, not a published artifact.
disable-model-invocation: true
argument-hint: [topic — e.g. "how does DNS work", "why is the sky blue", "what is a Merkle tree"]
---

# eli5

Explain like I'm someone who knows nothing about this topic, using a HTML artifact with big pictures and few words.

Topic: $ARGUMENTS

## Output

"HTML artifact" above means a **local, self-contained `.html` file**. Do NOT call the Artifact tool — nothing leaves this machine.

1. Write one self-contained file to `$HOME/.claude/eli5/<kebab-slug-of-topic>.html`, creating the directory if it doesn't exist. Inline all CSS and SVG; no external requests, no CDN links, no remote images.
2. Open it — `open <path>` on macOS, `xdg-open <path>` on Linux.
3. Reply with just the path, one line. The page is the answer; don't restate the explanation in the terminal.
