# RFCs and design docs

An RFC exists to get one decision made. Everything in it either supports the decision, specifies it, or plans it.

## Example

The user pasted the template `Context / Decision / Alternatives / Rollout / Risks`, and their notes say session reads add 40 ms p99 at peak and are 15% of primary reads. The notes don't give a TTL or a soak period.

```markdown
## Context
At peak traffic, session reads add 40 ms p99 to every request. They are
also 15% of all reads on the primary database.

## Decision
We will move the session store from Postgres to Redis. Sessions expire
after 24 hours. <!-- REVIEW: TTL isn't in the notes. 24h is assumed; confirm with the auth team. -->

## Alternatives
### Abandoned: add a Postgres read replica
A replica takes session reads off the primary, but the reads stay as slow.
We rejected it because latency is the main problem.

## Rollout
1. Write sessions to both Postgres and Redis.
2. Behind a feature flag, move session reads to Redis.
3. Stop writing to Postgres once reads have been stable for
   <!-- REVIEW: how long? --> days.
```

## Each section has one job

RFC sections map to different kinds of writing. Keep each to its kind; when content belongs to another kind, move it to that section and link.

- **Context / problem (explanation).** What is broken or missing, with the real numbers, symbols, and incidents from step 2. One bounded problem.
- **Decision / proposal (explanation, with a view).** State the recommendation in the first sentence ("We will ..."). Then why this option, with its trade-offs weighed, not listed. "It depends" is not a proposal.
- **Spec: API, schema, config (reference).** Describe only. Complete, dry, no persuasion. Mirror the structure of the thing: one entry per endpoint, field, or flag.
- **Rollout / migration (how-to).** Numbered steps, each ending in a state someone can check. Put the condition before the step ("If the backfill lags, ..."). No arguing inside the steps.
- **Risks.** Each risk with what happens, how likely, and what you do about it.

## Decision-doc rules

- **One proposal.** Every other option goes under Alternatives, titled `Abandoned: <option>`, with the reason it lost. A doc with two live options hasn't made its decision.
- **Decide, then flag.** When a choice needs someone else's input, make the best choice you can and mark it `<!-- REVIEW: ... -->` inline. Don't write an Open Questions section; unowned questions don't get answered. If the template has one, list only questions with a named owner there.
- **Too thin?** If the Proposal is shorter than Context plus Problem combined, the doc explains the problem better than it solves it. Expand the Proposal or cut the background.
- **Headings say the point** below the template's own headings: "Store sessions in Redis, not Postgres", not "Storage".
- **"We"** for the team's proposal and plans. "You" only in how-to steps addressed to an operator.

## Default sections (no template given)

1. Context
2. Proposal
3. Spec (only when the change has an interface, schema, or config)
4. Alternatives
5. Rollout
6. Risks
