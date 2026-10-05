# Jira tickets

The output is text the user pastes into Jira. Jira's markdown support varies by editor, so use plain labels, numbered lists, and bullets; no tables or nested formatting.

## Example

The user asks for a ticket for the pagination bug in `src/cart.py`, where line 3 is `return items[start:start + size - 1]`.

```
Title: Cart pagination drops the last item on every page

Observed: Each cart page shows 19 items. Items 20, 40, 60 and so on
never appear.
Expected: Each page shows 20 items, and every item appears exactly once.

Steps to reproduce:
1. Add 25 items to a cart.
2. Open page 1 of the cart.
3. Count the items: there are 19, and item 20 is missing.

Cause (from code): src/cart.py:3 slices items[start:start + size - 1].
A Python slice already excludes its end, so the "- 1" drops one item.

Acceptance criteria:
- With 25 items, page 1 shows items 1-20 and page 2 shows items 21-25.

Environment: [which environment did you see this in?]
```

## Title

The problem or the outcome, not the fix: "Cart pagination drops the last item on every page", not "Fix slice in paginate()".

## Bug

- **Observed** and **Expected**, each one or two sentences with real values.
- **Steps to reproduce**, numbered, each an action a stranger can do. End with what they will see.
- **Cause**, only as strong as the evidence. "Cause (from code)" when the code proves it, "Suspected cause" when it's a guess, and no cause line when there's no evidence. Never guess a cause and state it as fact.
- **Evidence**: file and line, log lines, error text, links the user gave.
- **Environment**, when it matters. Leave `[bracketed blanks]` for facts only the user has.

## Story or task

- Who needs it and why, in one or two sentences.
- **Acceptance criteria**: a bulleted list of checks that are true or false when the work is done. "Page 2 shows items 21-25", not "pagination works correctly".
- No implementation plan unless asked. That belongs in the PR or an RFC.

## Template

A ticket template the user pasted replaces these sections, the same way an RFC template does.
