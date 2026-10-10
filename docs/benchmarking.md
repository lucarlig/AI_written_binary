# Correctness and benchmarking

## Correctness first

Define the contract and independent reference before optimizing. Check ordinary inputs, zero-sized inputs where supported, boundary sizes, alignment, and invalid inputs according to the contract. Validate final output, return behavior, memory bounds, and relevant synchronization.

For floating-point work, fix input and accumulation precision, numerical tolerances, layouts, and allowed math approximations. A faster result with weaker semantics is a different experiment.

## Fair comparison

- Compare the same inputs, operation, precision, layout, and timing scope on the same hardware.
- Keep the baseline fixed and identify its provenance, version, settings, and artifact hash where available. Existing compiled or library baselines are comparison tools only.
- Separate computation timing from process startup, loading, allocations, transfers, and output. Report end-to-end timing separately when it matters.
- Warm up each variant. Use repeated samples and multiple measurement blocks; interleave or randomize variant order where practical.
- Select repetition counts and timing resolution appropriate to the workload. Record all samples and explain any exclusions. Do not select only the fastest run.
- For asynchronous GPU work, measure completed device work with appropriate synchronization and device timing. A host launch returning is not proof of completion.
- Specify whether buffers are reused and whether cache contents are warm. Apply the same policy to every variant.
- Check outputs outside the timed interval and ensure repeated work is actually executed.

## Required evidence

Record source commit, artifact SHA-256 hashes, generation and execution commands, OS, hardware model, relevant device features, driver/runtime versions, input sizes and seeds, warmups, repetitions, timing method, synchronization, and baseline settings. Record clock or power conditions and concurrent load when they materially affect the comparison.

Save a report, metadata, and raw samples in the function's `results/<run-id>/` directory. Report median and dispersion or percentiles, sample count, correctness outcome, and any limitations. Include throughput only with a stated unit and work definition.

Keep instruction bytes, artifact size, latency, throughput, and resource use as separate measures. A byte reduction is not evidence of a runtime improvement. If a difference is within observed variation, report no demonstrated gain; repeat or strengthen the experiment before claiming a speedup.
