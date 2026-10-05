# Perf

**First move.** Measure a baseline before changing anything: a fixed workload (same command, same input data, same machine, warm or cold stated), run at least 3 times. Reading code is not a measurement.

**Done means**
- The identical command after the change, at least 3 runs, reported as before and after with the median and spread.
- The delta is bigger than the run-to-run spread. If it isn't, the result is `inconclusive`, not a win.
- Correctness is unchanged: the same outputs or tests pass.
- The cause of the speedup is named (the loop removed, the query batched) and matches what a profile or timing breakdown shows.

**Allowed fallbacks**
- The real workload can't run locally: a micro-benchmark of the hot path, labeled `micro-benchmark`, with how it maps to the real workload.
- No stable machine (noisy laptop): more runs, and say the spread.

**At staging / prod**
- p50 and p95 latency, or throughput, for the touched endpoint or job in matched windows before and after the deploy. Name confounders: traffic shifts, other deploys in the window, cache warmup.

**Report must show**: the workload, baseline runs, after runs, delta with spread, and the artifact (profile, trace, raw timings) path.
