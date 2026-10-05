# Feature

**First move.** Write each claim the feature makes as something observable: "doing X on surface S shows Y". Two to five claims is typical. If the task came from an approved plan, its Verification section supplies them.

**Done means**
- Each claim is driven on its real surface and seen: response body and status for an API, snapshot or screenshot for a UI, stdout and exit code for a CLI, the emitted event or written row for a worker.
- At least one failure or edge path is driven (bad input, empty state, permission denied).
- Side effects are confirmed through a second, read-only view: the row in the database, the message on the topic, the file on disk.
- Existing tests still pass. New tests are welcome but don't replace driving the surface.

**Allowed fallbacks**
- Nothing can run locally: tests that call the public entry point, labeled `tests only`, and every claim they can't reach marked `not checked`.
- A surface the session can't drive (a device, a blocked browser tool): drive the layer beneath it and hand the user the exact manual check.

**At staging / prod**
- Staging: drive the claims only through reads and the profile's safe test actions, then run their cleanup.
- Prod: read-only evidence that the new path ran (a log line, span, or metric only the new code emits) and no new errors on the touched endpoints.

**Report must show**: each claim with the command or tool call that drove it and what came back, the edge path, and the side-effect check.
