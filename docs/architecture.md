# Repository architecture

Keep each function self-contained. Keep hardware-specific bytes separate from the function's meaning. Share only tooling that packages, loads, checks, or measures those bytes.

## Future experiment layout

This is a convention for future work, not a set of examples to create now.

```text
examples/<function>/
  SPEC.md
  tests/
  implementations/<target>/<variant>/
    README.md
    code.bits or code.hex
    layout.json
    out/                         Generated artifacts; ignored by Git
  results/<run-id>/
    report.md
    metadata.json
    samples.csv
```

`<function>` names the operation. `<target>` identifies the OS and native CPU or GPU architecture precisely. `<variant>` identifies a baseline or one optimization. `<run-id>` identifies a recorded measurement session.

### Function contract and tests

`SPEC.md` defines inputs, outputs, numerical or bit-level semantics, supported sizes, layouts, alignment, side effects, and correctness criteria. It also defines which work the benchmark includes.

`tests/` holds deterministic cases, edge cases, expected results, and independent reference checks. References are validation tooling, never a source of candidate executable bytes. Floating-point workloads must specify precision and tolerances before measuring performance.

### Native implementations

Literal bytes are the implementation source. `README.md` explains their encoding and the variant's hypothesis, dependencies, entry point, argument interface, memory ownership, and reproduction commands.

`layout.json` records the required binary layout: byte ranges, entry points, data, headers, metadata, and any documented offset patches. It must not become an instruction language or a hidden assembler. Omit unnecessary layout fields; do not invent a universal executable format.

An implementation may be a native executable, callable CPU function, or GPU module. Its documented packaging and loading path must preserve the authored instructions and avoid compilation or JIT. Generated artifacts go in `out/`; record their hashes in the results.

### Results

Keep reports beside raw samples and environment metadata. A report identifies the exact source commit, artifact hashes, correctness checks, baseline, commands, timing scope, and observed variation. No results directory is needed until a measurement actually runs.

## Shared tools

`tools/` owns literal-byte packing, format inspection, loading, test orchestration, timers, and result collection. Helpers cannot implement the measured function or emit instructions from symbolic source. Add a shared helper only when needed; keep function-specific oracles and workloads with their function.

## Target selection

Choose a function and exact target before creating its directory. Verify access to the hardware, native instruction encodings, binary container requirements, and loading interface before promising a runnable implementation.

For any future GPU work, record native device architecture and driver compatibility separately from host architecture. [NVIDIA's CUDA platform documentation](https://docs.nvidia.com/cuda/cuda-programming-guide/01-introduction/cuda-platform.html) distinguishes PTX intermediate code from native cubin binaries. A PTX-to-native compilation path fails this project's direct binary rule.

For NVIDIA targets, require on-target loading without off-target re-finalization of instructions. NVIDIA's binary compatibility guarantees do not cover manually generated or modified binaries; validate the exact device and driver rather than assuming compatibility. These distinctions are documented in the same [CUDA platform reference](https://docs.nvidia.com/cuda/cuda-programming-guide/01-introduction/cuda-platform.html#binary-compatibility).

The selected research direction is GEMM for LLM inference on NVIDIA Vera Rubin. Follow the [initial plan](gemm-inference-plan.md) before creating an implementation. The numerical contract and access to the target hardware remain unverified.
