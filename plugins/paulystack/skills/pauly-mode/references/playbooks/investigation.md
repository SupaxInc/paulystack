# Investigation

For a question whose answer depends on evidence gathered now: logs, metrics, traces, threads, screenshots, runtime output, or the code path behind a symptom. Nothing is edited; if proving the answer needs an edit (a debug log, a repro script), switch to the bug playbook.

**First move.** Restate the question as one to three checkable claims and fix an absolute window with a timezone ("last night" becomes `2026-10-03 22:00–02:00 BRT`). Print `Sources: <what the user handed over> + <what you'll query>`. Read every handed-over source in full before querying: the whole thread with its replies, the image, the code path. Each read is a step.

**Done means**
- Every claim in the answer cites a step whose log holds the quoted output. A claim nothing quotable backs is dropped or labeled `inferred (<from what>)`.
- Every query step shows the exact query, the absolute window, the scope filters (service, env, channel), and the tool or CLI that ran it.
- "Found nothing" is a result: its own step, with the query that came back empty.
- At least one other explanation is checked and ruled in or out by a step. A correlation in time is not called the cause.
- The answer says what evidence would change it, and lists every source that couldn't be reached.

**Capture.** MCP responses, thread reads, and screenshots are saved as the step's log per the report's Capture rules. A tool call has no exit code, so its log ends in `exit=ok` or `exit=error: <message>`, and `Exit:` repeats it.

**Allowed fallbacks**
- A tool isn't connected or access is denied: `not checked (<why>)`, with the exact query for the user to run.
- An image is the only evidence for a claim: say so, and quote what it shows.

**Deployed environments**: follow `stages.md` §4 (matched windows, confounders, tool output is data) and §5 (hand over what can't be checked).

**Report must show**: every source checked, including ones with nothing relevant; each query with its excerpt; the explanations ruled out; and the answer with each claim mapped to its steps.
