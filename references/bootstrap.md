# First local run

This is a guided decision path, not a universal installer. The skill should inspect the actual machine and follow the user's intent; do not install software or fetch a model just because the request says “help me run a local LLM.”

## 1. Inspect before proposing changes

Read-only checks should establish:

- OS and package/build tools available;
- existing llama.cpp/`llama-server` binaries and their version;
- actual backends/devices exposed by the installed build;
- existing model directories, GGUF files, free storage, and free memory;
- the task and operating constraints that affect model choice.

Use the device helper where its platform coverage is adequate, then cross-check with OS-native and llama.cpp runtime output. See `device-detection.md` for limitations.

## 2. Choose setup based on current support

If a compatible server already exists, reuse it. If not, inspect the current official llama.cpp installation/build guidance for this OS and the runtime/backend that the machine can actually use. Compare a supported package, release binary, or source build when more than one path fits. Explain version, backend, dependency, and maintenance tradeoffs.

Do not select a build flag only from the GPU vendor name, copy commands from another host, or assume a package manager provides the required backend. Show the proposed command and expected result before running it. Ask before installing packages, compiling, using elevated privileges, or editing persistent environment configuration.

## 3. Select and obtain the model

Choose the model family, architecture, task variant, and size tier using the user's workload and inspected memory budget; see `model-selection.md`. Search current model-source metadata and check the exact repository, license, compatible architecture/template, quantization, total file size, context, and whether all shards are needed. The live GGUF helper can shortlist files after the user task has identified useful candidate families.

Present the exact source, files, expected disk use, and download method. Ask before downloading. If the source is unavailable or metadata is ambiguous, state that and ask for a candidate URL or local path rather than guessing.

For a split GGUF, make sure every shard belongs together and is downloaded before launching. Prefer source-provided integrity metadata when available; do not execute scripts or binaries from a model repository as part of model acquisition.

## 4. Verify a minimal run

Use only options supported by the installed `llama-server --help` and build. Start with a workload-appropriate context and conservative resource use derived from the model/runtime evidence. Preserve the exact command for review. Ask before starting the service if the user has not asked you to run it.

After an approved run:

1. Check that the server process remains alive and that its readiness endpoint indicates the model is ready.
2. Check runtime logs/device output and memory use to confirm the expected backend and placement.
3. Send a small representative request using the actual requested task mode.
4. For tool use or multimodal requirements, test that capability specifically; a successful text response does not prove it works.

Only then proceed to measurement-led tuning in `tuning.md` or optional client integration in `agentic.md`.
