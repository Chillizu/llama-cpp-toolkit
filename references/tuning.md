# Measurement-led inference tuning

Tuning means adjusting inference settings for the selected model, actual llama.cpp build, hardware, and user workload. There is no universal best quant, context, thread count, offload value, or sampler profile.

## Establish a baseline

Before changing flags, record:

- exact model source, architecture, quantization, and llama.cpp version/build;
- actual device/backend and memory placement from runtime output;
- representative prompt, requested output length, context occupancy, and task result;
- load time, time to first token/prefill, decode rate, peak memory, and stability when available.

Use a repeatable workload that resembles how the user will use the model. Separate quality/capability checks from speed checks. Change one setting at a time and keep the previous working command so changes can be reverted.

## Select quantization and model placement

- Compare the exact model variants that are available and supported. A smaller quant saves weight memory but can reduce task quality; it is not guaranteed to decode faster.
- Do not describe one quant family as universally safe or fastest. Use current model/runtime/backend evidence and measure candidates on the target task.
- Choose GPU offload from the model's weight/KV budget, actual backend support, and observed free memory. Confirm placement after load; the requested offload value does not prove all layers fit or use the intended device.
- On shared/unified memory, account for the OS, desktop, and other active applications. On multi-GPU hosts, verify which devices receive weights and whether distribution is supported.

## Set context and KV cache

Choose context from the user's real prompt and session needs, the model's supported context/template, and available memory. Include system instructions, tool schemas, retrieved documents, conversation history, and generated output. Do not copy a fixed context tier or extend the model's supported context without verified runtime/model evidence.

KV-cache size depends on model architecture, context, cache type, and runtime implementation. Use the exact model metadata or runtime reporting. Quantized KV can save memory but support and quality/stability vary by architecture/backend/build; verify from current llama.cpp support and a representative multi-turn task before recommending it.

Check how the installed runtime interprets context and parallel slots. If multiple slots are in use, verify how the total context budget is divided rather than assuming each client receives the full configured value.

## Tune CPU and GPU use

- Read the installed binary's help and the host's CPU topology before choosing thread counts or affinity.
- Sweep a small range around values justified by physical cores, competing workloads, and runtime behavior. Use the same prompt and thermal state; keep a setting only if it improves the user's metric.
- Avoid pinning threads by a guessed P-core list or reducing OS memory management without evidence of a bottleneck.
- Treat backend-specific flags, serialization workarounds, and priority changes as opt-in fixes for observed conditions, not general defaults.

## Choose samplers and templates

First identify the model's expected chat template, reasoning/tool-call format, and the installed runtime's supported sampler options. Then choose a minimal configuration for the task:

- deterministic structured output or tool use may need low randomness and a compatible tool parser;
- open-ended writing may benefit from a different sampling range;
- penalties can help or damage output depending on the model and prompt.

Do not paste a canned sampler chain without checking the model template and `llama-server --help`. Test actual tool arguments, reasoning/content fields, and multi-turn output when those matter. Use the failure patterns in `stability.md` to diagnose observed regressions.

## Compare and stop

Report before/after values and the workload used. If a change improves speed but damages task quality, context, stability, or memory headroom, it is not an overall improvement. Stop tuning when changes no longer affect the user's target metric; consider a different model, quant, backend, or context plan instead of continuing to optimize irrelevant flags.
