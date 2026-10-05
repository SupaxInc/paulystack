# Refactor

**First move.** Pin current behavior before moving code: existing tests that cover the touched code (run them and confirm they touch it), a characterization test, or recorded outputs of the public entry points for a handful of inputs. Type check and lint are not a pin.

**Done means**
- The same pin passes after the change, run the same way.
- Public signatures, outputs, and side effects are unchanged. A change to any of them is named and handled under the Feature playbook instead.
- Nothing was left half-moved: old call sites migrated, the old path deleted or explicitly kept with the reason.

**Allowed fallbacks**
- No practical pin (no tests, can't run): an equivalence argument over the diff, labeled `inferred`, naming what would have made it checkable.

**At staging / prod**
- No new errors or latency change on the touched paths after deploy. Nothing more is needed.

**Report must show**: the pin (command and output before), the same command and output after, and any behavior change you found and split out.
