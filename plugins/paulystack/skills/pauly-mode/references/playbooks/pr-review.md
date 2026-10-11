# PR review

For a pr-recon run while pauly-mode is on: someone else's PR, reviewed read-only. Nothing is edited, posted, fetched, or checked out; the only writes are the report and its logs. It runs after pr-recon's step 6 Release, on the candidate findings, and before pr-recon anchors and drafts comments.

**First move.** Write each candidate as a claim: `<label> because <scenario>`, where the scenario is pr-recon's trigger → path → impact. Print `Sources: PR #<n> + <related PRs> + <tickets, docs, threads> + <services read>`.

**Done means**
- Every hop of each scenario (trigger, path, impact) cites a step whose log holds the line: a `git show <sha>:<path>` slice, a grep, or a read-only `gh` call.
- Every absence claim ("no caller checks the id", "nothing handles `PARTIAL`") is its own step, with the search that came back empty.
- Each `blocking` and `should fix` finding has at least one author's-side explanation ruled out by a step: the PR body says it's intentional, it's the same on base (`git show origin/<base>:<path>`), CI catches it, a related PR covers it, or something already enforces the merge order.
- Each `question` says what no read-only command could settle. A merge or deploy order is never left as a question: the steps show which side needs to be live first and what the other order breaks.
- Every PR on pr-recon's `Related:` line was read in a step.
- Each requirement line (`Met`, `Partial`, `Missed`, `Unclear`) cites the step that read its source (a ticket, doc, or thread, captured like an investigation's MCP step) and the diff step that meets it or shows the gap.
- Each claim about CI cites a step: `gh pr checks`, or `gh run view <id> --log-failed` for a failure.

**Capture.** The report's Capture rules, with every step read-only. Logs go under the review folder, never in the worktree.

**After the cross-exam.** A finding whose label call-saul's objection still stands against drops one label (`blocking` → `should fix` → `question`), or merges into the finding it shares a cause with. Then pr-recon continues from step 7 with the surviving labels.

**Report must show**: each finding with the steps behind every hop, what was ruled out, each requirement with its source and diff steps, and the related PRs and sources read.
