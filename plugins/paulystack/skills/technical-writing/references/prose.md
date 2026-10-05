# Prose rules

These apply inside every doc type. When a rule makes a particular sentence worse, fix the sentence another way or leave it.

## Example

Before:

> The check only fails on growth when the budget is exceeded. Additionally, it's crucial to note that the gate/ratchet ensures robust protection. We should probably maybe consider running it in CI.

After:

> The budget check fails only when the count grows past the budget. Run it in CI.

## Cut

- Words that do no work: "in order to" is "to"; "it is important to note that" is nothing; "additionally" at a sentence start usually goes.
- Hedging stacks: "could potentially possibly" is "may", or state it.
- Openers and closers: "Great question", "I hope this helps", "Let me know if", "In summary", "Overall".
- Generic conclusions: "This will greatly improve the developer experience." Say the specific result or cut it.
- Arguing with no one: "This isn't just a refactor" or "Contrary to what you might think" when nobody claimed otherwise.
- Writing about the previous version: "Now uses Redis instead of Postgres" in a doc that describes current behavior. Describe what is. Mention the old way only in changelogs and migration guides.

## Plain words

- The short, everyday word: "use", not "utilize" or "leverage"; "help", not "facilitate"; "is", not "serves as" or "stands as".
- No AI vocabulary: delve, crucial, pivotal, robust, seamless, comprehensive, landscape, tapestry, testament, underscore, showcase, foster.
- No "not just X, but Y". State Y.
- Use the natural number of items. Don't force lists or phrases into threes.
- No invented jargon or metaphor nouns (ratchet, substrate, north star, flywheel). Name the mechanism.
- Specific over vague: "a column rename fails the build", not "schema changes can cause issues". The specifics come from step 2's facts, never from imagination.

## One reading only

- Put "only" and "not" next to the word they limit: "fails only on growth" and "only fails on growth" differ.
- Every "it", "this", "they" points at one obvious noun. Repeat the noun when in doubt. Never "this" for a whole previous sentence.
- Break noun stacks: "the proto import budget check script" is "the script that checks the proto-import budget".
- Don't drop verbs: "Phase 1 moves the converters and Phase 2 the runtime" gives Phase 2 no verb.
- One name per thing, everywhere. If it's "the budget check" once, it's never "the gate" later.
- No slashes for "or": write "a, b, or both", not "a/b" or "and/or".
- Say who does what: "the compiler checks types", not "types are checked", unless the actor doesn't matter.

## Punctuation and rhythm

- No em dashes. Use a period or a comma. Prefer periods over semicolons.
- Arrows (`→`) only in diagrams, not in sentences.
- Bold only for a label that starts a list item and is followed by new detail, not for emphasis inside sentences.
- Sentence case for headings.
- Vary sentence length. Short sentences land a point; a longer one can carry a fact with its condition. Split a sentence that carries two ideas, not one that is merely long.
