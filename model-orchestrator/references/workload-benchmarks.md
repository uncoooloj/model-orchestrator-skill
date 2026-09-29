# Comparable Workload Benchmarks

Use this reference when a model or runtime comparison is meant to inform routing for an existing workflow, especially when cost or throughput is part of the decision. Keep the evaluation bounded and return to the user’s actual deliverable when the bound is reached.

## Protocol

1. Freeze the workload before running candidates:
   - task cases and expected outputs;
   - prompts, system instructions, and other assets;
   - the action and tool contract, including discovery, schemas, permissions, and failure behavior;
   - the scoring rubric, pass thresholds, and critical-error definitions.

2. Record the harness for every result. Include the candidate identifier, version or configuration, reasoning and sampling settings, context limits, tool availability, retries, timeout, concurrency, and the evaluator version. Record any justified variant separately with its reason; do not silently change a frozen input, tool path, or budget between candidates.

3. Preserve attempt-level outcomes. Report completion and quality separately, and count critical errors and timeouts independently. State each denominator: keep the full scheduled-case denominator for coverage, and report quality on returned answers as a conditional measure. A completed case can still fail quality; a timeout has unknown answer quality and must not be counted as a quality pass or failure.

4. Keep recovery separate from first attempts. Record the trigger, recovery action, and recovery result in a separate field or table. Do not replace a first-attempt failure with a later successful retry when reporting initial completion.

5. Separate usage from monetary cost. Mark usage as `measured` only when run telemetry records it. Mark monetary cost as `measured` only when billing confirms the charge; a cost derived from measured usage and a price assumption is still `estimated`. Mark either field unavailable when it cannot be supported, and keep units and assumptions with the result.

6. Do not rank candidates across different execution contracts. An inline smoke test with discovery or action tools disabled is harness-confounded with a tool-loop result. Report the observations and the incompatibility; do not turn the differing scores into a controlled model ranking. If material harness metadata is missing, label the comparison observational or inconclusive. Re-run under the same contract if a controlled ranking is needed.

7. Predeclare the evaluation bound: cases, attempts, wall-clock limit, and cost or usage ceiling. Stop at the first bound reached, report what was and was not covered, and return to the actual user deliverable. A benchmark is evidence for a decision, not a replacement for the workflow it was meant to improve.

8. Gate unattended adoption on a supervised live pilot. Exercise the required connector, action, and restart or recovery path with an operator observing the run. Before unattended side effects, verify the intended action permissions, duplicate prevention, bounded retries, audit records, and recovery behavior. Keep automatic actions disabled until the relevant acceptance checks pass and the user has authorized that action scope. Neither a benchmark score nor a successful pilot grants new permission.

## Synthetic comparison fixture

This small fixture is entirely fictional and is suitable for checking that a comparison preserves its boundaries. It contains no real model, price, budget, incident, customer, or private case.

Frozen workload: four synthetic tasks (`alpha-sort`, `beta-summarize`, `gamma-route`, `delta-extract`), one fixed prompt pack, one fixed output rubric, and one tool-loop contract with discovery plus one action. Each candidate receives one first attempt per task; recovery is recorded separately.

| Candidate | Harness | First attempts | Completed | Quality passes | Critical errors | Timeouts | Recovery | Usage evidence | Monetary-cost evidence |
| --- | --- | ---: | ---: | ---: | ---: | ---: | --- | --- | --- |
| `candidate-inline` | inline smoke; tools disabled | 4 | 4 | 3 | 0 | 0 | 1 recovery after a quality failure, recorded separately | measured | estimated |
| `candidate-tools` | tool loop; frozen contract | 4 | 3 | 2 | 1 | 1 | 1 recovery after a timeout, recorded separately | measured | measured |

Quality passes are conditional on completed answers; coverage uses all four scheduled cases, and the timeout’s answer quality is unknown. The fixture does not support a controlled rank: the harnesses differ, and the inline run cannot exercise the frozen action contract. A valid next step is to repeat both candidates under the same contract within the predeclared bound. If that bound is exhausted, preserve this comparison as limited evidence and return to the deliverable that motivated it.
