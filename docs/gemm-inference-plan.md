# GEMM for LLM inference: initial plan

Research checked on 2026-10-10. This is a proposed sequence, not a runnable kernel, fixed numerical specification, or performance result.

The selected direction is native GEMM for inference on Rubin GPUs in the NVIDIA Vera Rubin platform. Start with one GPU and one precisely defined operation before investigating rack-scale serving.

## What we can establish

- No public NVIDIA Rubin native-binary emulator or validated performance simulator was found. NVIDIA's [CUDA-GDB documentation](https://docs.nvidia.com/cuda/cuda-gdb/) describes debugging on actual hardware. This finding does not rule out internal or partner-only simulators.
- The third-party [Accel-Sim project](https://github.com/accel-sim/accel-sim-framework) currently advertises validated Hopper support and hardware-derived SASS trace replay. It is not evidence that a newly authored Rubin binary executes correctly or achieves a measured Rubin speedup.
- NVIDIA's [CUDA Binary Utilities](https://docs.nvidia.com/cuda/cuda-binary-utilities/) document native instruction mnemonics, disassembly, and cubin's ELF container. The public reference is not a complete native instruction bit-encoding and kernel metadata specification for authoring a Rubin module from scratch.
- The [CUDA platform guide](https://docs.nvidia.com/cuda/cuda-programming-guide/01-introduction/cuda-platform.html) explains PTX compilation, native cubins, architecture compatibility, and driver re-finalization. NVIDIA does not support manually generated or modified native GPU binaries under its compatibility guarantees. The candidate must load on its exact target without PTX compilation or instruction re-finalization.
- NVIDIA [announced CoreWeave Rubin NVL72 availability on September 30, 2026](https://blogs.nvidia.com/blog/coreweave-agentic-ai-vera-rubin/), including early-access customers. This is a lead for hardware access, not verification of capacity, account eligibility, pricing, or access to raw kernel launches.

## Workload selection

We have no verified production GEMM trace, matrix dimensions, numerical formats, or sharding configuration for Claude Fable or GPT-6 Astra. Their public [Fable page](https://www.anthropic.com/claude/fable) and [Astra model page](https://developers.openai.com/api/docs/models/gpt-6-astra) do not supply those serving contracts. Use an accessible open-weight model or an explicitly labeled synthetic workload, and never describe it as an exact reproduction of either service.

Measure prompt processing (prefill) and token generation (decode) separately. NVIDIA discusses their different compute and memory behavior in its [inference analysis](https://developer.nvidia.com/blog/co-designing-ai-model-attention-for-fast-interactive-long-context-inference/). Batch size, context, prefix caching, and speculative decoding can change the bottleneck; profile rather than assuming it.

For a projection `C[M,N] = A[M,K] * B[K,N]`, `M` is the number of processed token rows, `K` the input width, and `N` the output width. Use observed projection dimensions and per-device sharding from the chosen workload. A large square GEMM alone is not an inference workload.

| Family | Inputs to capture | Candidate optimization hypothesis |
| --- | --- | --- |
| Prefill projections | Prompt chunk size, batch, projection dimensions | Improve Tensor Core utilization and overlap data movement |
| Decode projections | Active batch and small token-row count | Improve weight access, tiling, and launch overhead |
| MoE expert operations, if applicable | Tokens per expert and routing distribution | Group uneven small GEMMs and reduce intermediate traffic |

These are hypotheses. Select the first one only after profiling a real workload and a strong baseline.

## Sequence and verification gates

1. **Verify target access and encoding feasibility.** Record the actual GPU architecture, driver, module format, argument ABI, and evidence for instruction encodings and metadata. Identify missing information. Access to a managed inference API alone is insufficient; we need native module loading and profiling. Hardware from another architecture is a separate development target, with separate bytes and results.
2. **Establish the native loading path.** Author a minimal native module entirely from literal bytes and show that it executes and writes a known result through the driver. This is a prerequisite check, not a separate benchmark example. Verify artifact bytes and hash, no embedded PTX, no JIT, and no off-target re-finalization. If we cannot verify this path, continue workload/reference work and report the limitation; do not present compiled output as a candidate.
3. **Capture representative shapes and a baseline.** Record `M`, `N`, `K`, layouts, strides, precision, accumulation, epilogue, batch distribution, and any grouping. Benchmark prebuilt cuBLASLt with several suitable algorithms selected and timed outside the measurement loop; record workspace and algorithm choices. NVIDIA's [cuBLAS tuning guidance](https://developer.nvidia.com/blog/introducing-grouped-gemm-apis-in-cublas-and-more-performance-updates/) explains why a default matmul call alone is not the strongest baseline. Use an appropriate grouped baseline if we later choose MoE.
4. **Fix one initial numerical contract.** Proposed starting point: BF16 inputs and weights, FP32 accumulation, and an explicitly chosen output type and tolerance. Fix layout and one workload family before writing a GEMM candidate. Keep the independent oracle in the test harness. Add FP8 or NVFP4 only as a separate contract with scaling and accuracy requirements, not as an unqualified speed comparison.
5. **Build a correct native GEMM.** Start with a small supported shape, then test workload dimensions and boundary handling. Validate against an independent numerical reference. Prove Tensor Core instruction encodings and synchronization separately before relying on them for performance. Scalar correctness is a milestone, not a competitive inference result.
6. **Optimize one bottleneck at a time.** Preserve the verified baseline. Test tiling, data layout, register use, memory staging, synchronization, and overlap. Investigate fused output operations only when they belong to the workload, and compare against equivalent fusion in the reference path.
7. **Prove inference benefit.** Report device execution time, raw samples, numerical error, and throughput under a defined workload. Then integrate the winning candidate into an accessible inference workload and measure time to first token, time per output token, throughput, and tail latency at fixed concurrency and quality. Measure attention, routing, and communication too; a faster GEMM is not automatically a faster model. Add multi-GPU communication and NVL72-scale claims only after single-GPU evidence exists.

## Repository use and current status

Once the first numerical contract and target are verified, create `examples/gemm/` using [the repository layout](architecture.md), with separate native target/variant directories and recorded results. Keep the launcher and timers in `tools/`; candidate arithmetic stays in authored bytes. Reference libraries are comparison tools only.

Completed: public-source feasibility research and this plan. No GPU module, GEMM candidate, emulator, native encoding proof, Rubin hardware access, or benchmark result has been produced or verified.
