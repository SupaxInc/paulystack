# READMEs and docs pages

## Example

Adding a skill to a README whose Skills table already has rows like `| name | what it does, how to invoke it, where its files live |`:

```
| `technical-writing` | Writes and reviews RFCs, PR descriptions, commit messages, Jira tickets and READMEs. Gathers facts from the code before writing and marks unknowns with `REVIEW` instead of guessing. Outputs text to paste; it doesn't create tickets. Model-triggered, or `/paulystack:technical-writing [doc type or path to a draft]` |
```

The row copies the table's existing format. A cold reader asking "Does it post to Jira?" found no answer in the first draft, so "Outputs text to paste" was added.

## One kind of writing per section

Decide what each section is for, and keep it to that:

- **How-to** (install, setup, "how to add X"): numbered steps toward one goal. Assume a competent reader. Put conditions before steps. The exact command in a code block, and what the reader should see when it worked.
- **Reference** (tables of commands, keybindings, config options, file mappings): describe only. Complete and dry. Mirror the structure of the thing. No "why" paragraphs inside a table; move them to their own section and link.
- **Explanation** (why it's built this way, design choices): prose, allowed to hold a view.
- **Tutorial** (a guided first project): rare in a README. Only when the repo has true beginners.

When one section mixes these, split it.

## Match the doc you're editing

- Before adding to an existing doc, read the neighboring sections and copy their format: table columns, heading levels, how invocations and paths are written.
- Use the real command, flag, symbol, and path names from the repo, in code font. Check each one exists.
- Don't reword sections you weren't asked to change.
