# Agent instructions

## Mission and scope

Investigate whether native binaries authored directly by agents can improve measured performance. Preserve correctness and reproducibility. A negative result is useful evidence.

Current direction: GEMM for LLM inference on Rubin GPUs in the NVIDIA Vera Rubin platform. Follow [docs/gemm-inference-plan.md](docs/gemm-inference-plan.md). The exact numerical contract, native encoding/loading path, and access to Rubin hardware remain unverified. Do not add unrelated examples.

## Direct binary rule

- Author executable instruction bytes from scratch as literal bits or hex bytes. Keep them readable and version controlled.
- No compiler, assembler, linker, JIT, transpiler, symbolic instruction encoder, or code-generation library in the candidate build or execution path.
- Do not compile or assemble source and relabel its output as directly authored binary. Do not copy executable code from compiler output, libraries, or existing binaries into a candidate.
- Scripts may decode literal bytes, write documented headers and metadata, package data, load the binary, validate it, and benchmark it. They must not translate program logic or instruction mnemonics into machine code.
- OS and driver services may manage memory, files, loading, and launches. The computation under test must execute in the authored bytes, without delegation to an existing implementation.
- Disassemblers, debuggers, profilers, and existing reference implementations are allowed for inspection, correctness, and comparison. Reference code and binaries must never supply candidate instruction bytes.
- GPU candidates must contain native instructions for the declared device. PTX compiled by a driver JIT does not meet this rule. Record the loading path and verify that it performs no candidate compilation or off-target instruction re-finalization.

## Repository boundaries

- Follow [docs/architecture.md](docs/architecture.md). Each function owns its contract, tests, implementations, and results under `examples/<function>/`.
- Keep targets and optimization variants separate. Preserve a verified baseline before experimenting.
- Shared helpers belong in `tools/` and must contain no workload implementation. Keep helpers minimal; add them when an experiment needs them.
- Never introduce a compiler as a workaround. If native encoding, container documentation, or target hardware is unavailable, record the limitation and continue only with work that can be verified.
- Keep edits within the requested function or infrastructure task. Do not change unrelated experiments.

## Working sequence

1. Declare the function contract, target CPU or GPU, instruction set, OS, ABI or launch interface, binary format, dependencies, and required device features.
2. State one optimization hypothesis and its success criterion before changing bytes.
3. Document instruction boundaries, operands, byte order, offsets, entry points, memory use, and relevant register or launch constraints.
4. Verify the final emitted bytes against their literal source. Record the artifact hash and exact generation command.
5. Check observable behavior against independent references, including edge cases. Respect calling conventions, memory bounds, and synchronization.
6. Benchmark verified candidates under [docs/benchmarking.md](docs/benchmarking.md). Retain raw samples and report uncertainty.

Use primary documentation for encoding, ABI, container, driver, and hardware claims. Distinguish documented facts from assumptions and untested hypotheses. Static inspection alone does not prove execution or performance.

## Reporting and commits

- State what changed, why, what actually ran, and the result. Do not claim speedup from a smaller file, fewer bytes, fewer instructions, a single timing, or a different workload.
- Mark unrun work and unavailable hardware explicitly. Never present another device's results as results for the requested target.
- Use `git commit -s`. Do not bypass verification or hooks.
