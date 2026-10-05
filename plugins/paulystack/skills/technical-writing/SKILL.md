---
name: technical-writing
description: 'Writes and reviews engineering prose: RFCs and design docs, PR descriptions, commit messages, Jira tickets, READMEs and repo docs. Gathers real facts from the code and the user before writing, shapes the doc by its type, cuts AI tells and padding, and has a fresh subagent read long docs cold to find gaps. Use when the user asks to write, draft, tighten, or review an RFC, design doc, proposal, PR description, commit message, Jira ticket, bug report, user story, README, or docs page, such as "write a PR description", "draft an RFC for X", "commit msg for this", "write a ticket for this bug", or "review my design doc".'
argument-hint: '[doc type or path to a draft to review]'
---

# Technical writing

Request: $ARGUMENTS

Not for code comments (the repo's comment rules cover them), UI copy, or chat answers. Use your normal approach there.

## 1. Pick the type

Read the one reference file for the doc type, plus [references/prose.md](references/prose.md) for every type except commits:

| Type | Reference |
|---|---|
| RFC, design doc, proposal, ADR | [references/rfc.md](references/rfc.md) |
| PR description, commit message | [references/pr-and-commit.md](references/pr-and-commit.md) |
| Jira ticket: bug, story, task | [references/ticket.md](references/ticket.md) |
| README, docs page, guide | [references/readme.md](references/readme.md) |

A commit message needs only `pr-and-commit.md`. Reviewing the user's own draft is step 6, using the reference for the draft's type.

## 2. Gather facts; never invent them

Read what the doc is about before writing a word of it: the code, the diff, the user's notes, logs, or tickets they point at. For a PR or commit, read the whole change against its base (`git diff <base>...HEAD`, or `--staged` for a commit), not only the last commit.

Every number, name, symbol, path, date, and owner in the doc comes from something you read or from the user. When a sentence needs a fact you don't have:
- ask for it, when the user is there to answer and the fact changes the doc's point;
- otherwise write the sentence without it, or in an RFC mark the assumption `<!-- REVIEW: what is assumed and who should confirm -->`.

A doc with a visible gap is fixable. A doc with a plausible invented number gets approved wrong.

## 3. Use the user's template

If the user pasted or pointed at a template in this conversation, its sections and headings replace the reference file's default section list. Keep its order and names. The reference file's rules still apply inside each section. Nothing is saved; a template applies to the conversation it was given in.

## 4. Draft

Write to the reference file and `prose.md`. Write the doc where the user asked for it; with no location given, print it in chat, fenced when it will be pasted somewhere (PR body, commit, Jira).

## 5. Cold-reader check (RFCs and READMEs only)

The writer can't see gaps its own context fills in. For an RFC or README:

1. Write 3 to 5 questions the doc's target reader will bring to it (for an RFC: a reviewer from another team; for a README: someone who just cloned the repo).
2. Dispatch one subagent with only the draft's full text and the questions. Tell it to answer each question from the draft alone, quoting the line it used, and to say "not in the doc" when the draft doesn't answer it. It inherits nothing else from this session.
3. For each wrong or missing answer, fix the draft: add the fact, or a `REVIEW` marker if you don't have it.
4. Re-run the check once on the fixed draft. Report any question still unanswered.

## 6. Review mode

When the user asks you to review their draft, don't rewrite it and don't edit the file. Report issues most severe first, one self-contained block each:

```
1. <High|Medium|Low>: <short name>
   "<quoted text from the draft>"
   <what's wrong, in one or two sentences>
   Fix: <the concrete rewrite or action>
```

Severity: High changes what a reader understands or decides (ambiguity, a missing decision, a wrong or unsupported claim, a missing section the reader needs). Medium costs the reader time (wrong structure, two names for one thing, hedging). Low is polish (filler, AI tells). List only issues you are confident about; skip taste. End by offering to apply the fixes. Edit only when the user says to.
