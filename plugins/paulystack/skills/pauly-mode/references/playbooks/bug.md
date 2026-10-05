# Bug

**First move.** See the failure before editing, on the lowest stage that shows it: a local run from the profile, else a failing test through the public entry point (the way users call it), else read-only staging or prod signals carrying the error's signature. Save the exact command; it is the repro.

**Done means**
- The same repro, on the same surface, fails before the fix and passes after. Both outputs are in the report.
- The root cause is named at `file:line`, with the observation that confirmed it (a log line, a value you printed, the input that triggers it), not a guess from reading code.
- When it's cheap, one neighboring input of the same class also passes (the other operator, the empty case, the retry path).
- Existing tests still pass.

**Allowed fallbacks**
- Can't reproduce: say so, record what you tried, and label the fix `inferred`.
- A unit test on the fixed function shows that branch behaves; it doesn't show the reported symptom is gone. It supports the repro, never replaces it.

**At staging / prod**
- Staging: rerun the repro there only if it is a read or one of the profile's safe test actions.
- Prod: compare the error signature (issue events, error-log count, failing endpoint's error rate) in equal windows before and after the deploy, with traffic noted.

**Report must show**: the repro command, failing output, the root cause line, passing output from the same command, and any neighbor input you checked.
