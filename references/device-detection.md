# Inspect the machine

Hardware discovery supports a model recommendation; it does not by itself prove a runtime backend works or determine the best model. Gather the strongest current evidence available and label estimates.

## What to inspect

- OS and relevant runtime/build tools.
- CPU model, physical/logical cores, and architecture when the runtime depends on them.
- Total and currently available system memory.
- Every relevant GPU: model, dedicated or shared/unified memory, and currently free memory where readable.
- Driver/runtime availability and the actual devices exposed by the selected inference engine.
- Existing `llama-server` binaries, build/version, model directories, and other processes using memory.

First check for the user's existing runtime and model files. A working local runtime is more useful evidence than a backend guess based on GPU vendor. For llama.cpp, use the installed binary's `--list-devices` and startup/load output when available. Cross-check reported memory with OS-native tools if the result looks incomplete or implausible.

## Helper script coverage and limitations

`scripts/detect-device.sh` is a read-only helper for Linux and macOS environments with the expected system tools. It reports an OS, a preferred GPU adapter, memory estimates, CPU counts, notes, and a `suggested_backend`. The suggestion is only a starting point:

- Hybrid/multi-GPU selection is heuristic. Ask which GPU the user intends to use and verify the actual runtime's device list.
- `nvidia-smi` and `vulkaninfo` can expose memory details when installed; absence of a value does not mean a GPU is unusable.
- macOS available RAM from this helper is a coarse estimate. Confirm it with current system monitoring when fit is close.
- Some Linux GPU memory layouts and drivers require native/vendor tools not covered by the helper.
- On Windows, the script does not provide a complete hardware inventory. Use PowerShell or the installed runtime's own diagnostics; do not treat its Windows label as complete Windows support.
- CPU efficiency-core counts are estimates. Do not use them to pin threads without checking CPU topology.

The helper emits a human summary and a JSON object similar to:

```json
{
  "os": "linux",
  "gpu_vendor": "intel",
  "gpu_model": "...",
  "gpu_memory_gib": null,
  "memory_type": "shared",
  "ram_gib": 30.0,
  "ram_available_gib": 9.6,
  "cpu_cores": 22,
  "cpu_physical": 16,
  "cpu_efficient": 8,
  "suggested_backend": "vulkan",
  "notes": []
}
```

Interpret `gpu_memory_gib: null` together with `memory_type` and `notes`; it can mean shared memory or an unreadable dedicated-memory value. Do not count total unified/system RAM as entirely available to model weights.

## Select and verify a backend

Do not turn a vendor-to-backend table into an automatic build instruction. The viable backend depends on the OS, GPU generation, driver, installed llama.cpp build, and user preference. Use this sequence:

1. Inspect an existing `llama-server` and its version before proposing a new build.
2. Read the runtime's current `--list-devices`, build features, and startup output.
3. If no usable runtime exists, compare currently supported install/build paths for the detected OS and GPU, and explain tradeoffs.
4. Present the proposed package/build command before execution. Ask before installing dependencies, compiling, or changing persistent system configuration.
5. After setup, verify the device with the runtime itself and perform a small load before claiming GPU offload works.

Backend names such as CUDA, ROCm/HIP, Vulkan, Metal, and CPU are possibilities to check against current runtime support, not a guaranteed mapping from GPU vendor. Use the backend actually exposed by the installed build.
