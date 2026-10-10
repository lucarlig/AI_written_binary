# AI_written_binary

An experiment in agents writing native machine-code binaries from scratch, then measuring whether direct control over instructions and binary layout improves performance.

Candidate programs use no compiler, assembler, linker, or JIT. Their source of truth is explicitly authored instruction bytes. Small scripts may package those bytes, load them, check correctness, and measure execution. Existing libraries may provide independent reference results and performance baselines; they must not perform the candidate's computation.

Performance improvement is a research question, not an assumed result. Report correctness, execution time, throughput, and binary size separately, including experiments that show no gain.

## Structure

```text
AGENTS.md             Rules for agents working in this repository
docs/
  architecture.md     Layout and boundaries for future experiments
  benchmarking.md     Correctness and measurement requirements
examples/             One independent directory per function, when selected
tools/                Shared byte packaging, loading, and measurement helpers
```

Read [AGENTS.md](AGENTS.md), then the [architecture](docs/architecture.md) and [benchmarking rules](docs/benchmarking.md).

## Status

Current direction: GEMM for LLM inference on NVIDIA Vera Rubin. Read the [initial plan and feasibility findings](docs/gemm-inference-plan.md).

No examples, implementations, or benchmark results are included yet. The first numerical contract, native binary loading path, and Rubin hardware access still need verification.
