---
name: llama-cpp-toolkit
description: Help users choose, set up, run, and tune local GGUF language models with llama.cpp based on their goals and actual hardware. Use for local model family or size selection, hardware-aware model fit, llama.cpp backend setup, GGUF discovery, inference tuning, llama-server troubleshooting, or connecting a local OpenAI-compatible endpoint. Keep recommendations current and environment-specific; do not assume a fixed model, GPU backend, or launch command.
---

# Local LLM Toolkit: llama.cpp and GGUF

An agent-guided path from a user's goal to a verified local model run. Start with what the user wants to do and what their machine can run, then discover current model choices and build a plan around the actual environment. The bundled scripts are optional helpers for repeatable checks and live GGUF metadata; they are not a required execution path or a substitute for inspecting the installed runtime.

This toolkit currently focuses on `llama.cpp` and GGUF. It can discuss other local runtimes when useful, but must clearly distinguish guidance from tested support.

## Use this skill when

- The user wants to run a model locally, choose a model for a task, or assess whether a model fits their machine.
- They need help inspecting CPU, GPU, memory, drivers, an existing llama.cpp build, or downloaded model files.
- They want to set up or troubleshoot `llama.cpp` / `llama-server`, tune inference, or connect an OpenAI-compatible client.
- They want a repeatable personal manager based on the optional `llmctl` template.

## Workflow

1. **Understand the intended work.** Find out the task and the constraints that change model choice: for example coding or chat, tool use, vision, context length, response speed, quality, privacy, offline use, and available storage. Ask only the missing questions that materially affect the recommendation. Do not lead with a fixed questionnaire when the request already answers them.
2. **Inspect the machine and existing setup.** Prefer read-only inspection first. Identify OS, CPU, available system RAM, GPU model and usable memory, driver/runtime state, existing inference engines, `llama-server`, and local model locations as relevant. Use `scripts/detect-device.sh` when it fits the environment; treat its backend as a hint and verify against the actual runtime's device list or equivalent. State gaps and uncertainty instead of inferring a working backend from the GPU brand alone.
3. **Choose model candidates from current evidence.** Translate the user's task into model capabilities, then search current sources for suitable families, architectures, size tiers, and exact variants. Compare a small set of candidates on task ability, tool or modality support, context, license, quantization, download size, and measured or documented quality/speed where available. Do not use a fixed favorite-family list or a universal parameter-count threshold. Model size alone does not establish quality or tool capability.
4. **Estimate fit for this machine.** Use current available VRAM or shared/system memory, model weight sizes, architecture-specific KV-cache requirements at the requested context, runtime/backend overhead, and observed headroom. Distinguish dedicated VRAM from shared memory. Give a range or confidence label when metadata or free-memory measurements are incomplete; do not call a model a guaranteed fit from weight size alone or rely on a universal fixed buffer.
5. **Explain the source and obtain the exact artifact.** Give the repository/source, license, exact GGUF file or split-file set, quantization, total size, and any access or network requirements. Check that the candidate is actually compatible with llama.cpp and that all shards are accounted for. The live Hugging Face helper can support GGUF discovery; if it is unavailable, search current model-source metadata and say when a claim could not be verified. Do not download anything unless the user asks to obtain it.
6. **Set up only what is missing.** Reuse an existing compatible runtime and model when possible. If setup is needed, select instructions for the detected OS, hardware, driver state, and user preference, using current project documentation. Explain the intended changes. Proceed when the user clearly requested those actions; ask only when the package, model source, persistent change, or service scope is materially ambiguous. If the user only requested advice, stop at a practical recommendation and next steps.
7. **Make a minimal first run and verify it.** Build the launch command from the installed binary's supported options and the chosen model's metadata. Avoid transplanting flags from another machine. Verify the actual backend, server readiness, one representative prompt, and the user's required capability (for example, tool call or vision) before claiming success.
8. **Tune from a baseline.** Measure the user's representative workload first. Change one setting at a time and compare the result against their quality, latency, memory, and throughput goals. Only recommend options supported by the installed build; derive ranges from the detected hardware and model metadata rather than fixed context/threads tables.
9. **Offer optional integrations after the first run.** Connect a coding agent or other client only if requested. Use the local OpenAI-compatible endpoint and confirm the client is using it. If recurring model management could help, ask whether the user wants an `llmctl`-style manager or just advice; add only the capabilities they choose. Treat process management and router mode as optional workflows, not prerequisites for choosing or running a model.

## Operating rules

- Separate **recommendation** from **execution**. Hardware inspection can be read-only; package installation, model downloads, persistent edits, and service lifecycle changes require clear user intent. A request for a recommendation alone does not authorize them.
- Prefer explanations and commands tailored to the current environment over a universal installer or a hard-coded launch profile. Use scripts only when they provide reliable evidence or remove repetitive work.
- Treat device-detector output as an estimate. The runtime's device list, actual load behavior, and measured free memory are stronger evidence.
- Search live model-source metadata for current candidates. Verify the exact architecture, task variant, quant, context, license, file set, and source before recommending a download.
- Do not present file size as a full memory estimate. Include model-specific KV cache, context, runtime/backend overhead, and uncertainty.
- Avoid absolute size cutoffs for coding, tool use, or other capabilities. Check current architecture support and evaluate representative tasks on the actual runtime when capability matters.
- Do inference-time tuning only. This skill does not train or fine-tune model weights.

## Optional scripts

| script | purpose | limits |
|---|---|---|
| `scripts/detect-device.sh` | Read-only hardware summary and JSON for supported shells/OS tools | Heuristic; verify GPU/backend/memory against the installed runtime and OS-native tools |
| `scripts/recommend-models.sh` | Live Hugging Face GGUF sibling/file-size search with a KV estimate | GGUF-focused; metadata and budget are estimates, not a full compatibility or quality evaluation |
| `scripts/llmctl-template.sh` | Starting point for personal `list/start/stop/status/profile/client` management | A template to adapt; it is not a fully configured manager and is not needed for the core workflow |

## References (load when useful)

- `references/device-detection.md` — device evidence, limitations, and backend verification.
- `references/model-selection.md` — workload-led family/architecture search, exact source checks, and fit estimation.
- `references/bootstrap.md` — setup choices after inspecting the existing machine.
- `references/tuning.md` — measurement-led inference tuning.
- `references/stability.md` — common llama.cpp failures and diagnosis.
- `references/agentic.md` — local OpenAI-compatible client setup and capability verification.
- `references/build-your-own-manager.md` — optional manager design and local `llmctl` patterns.

## Script requirements

The shell helpers require bash, `curl`, and `python3`; `jq` is optional for device-report formatting. Hardware detection depends on OS-specific utilities and is not equally complete on every platform. When a helper is unavailable or incomplete, use native inspection tools or ask the user for the missing diagnostic output.
