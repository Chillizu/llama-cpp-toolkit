# Model discovery and fit

Choose for the user's task and machine from current source evidence. This is a decision process, not a permanently maintained list of favored model families, fixed size cutoffs, or one universal memory formula.

## 1. Define the workload

Translate the user's request into capabilities and constraints. Ask only about gaps that affect the result. Common decision points include:

- chat, coding, tool use, vision/audio, structured output, or another concrete task;
- preferred language and license constraints;
- quality versus response speed, and whether low latency matters more than peak quality;
- context length or prompt size, number of concurrent users, and session length;
- privacy/offline requirements, network access for downloads, and disk space.

Do not assume that a model described as "instruct" has the required tool, vision, or structured-output support. Verify the exact architecture, model variant, tokenizer/template, and llama.cpp support for the needed capability.

## 2. Inspect the available machine budget

Use current measurements and installed-runtime evidence:

- OS, CPU and memory available to the process;
- dedicated GPU memory, shared/unified memory, or multi-GPU distribution;
- currently free memory and other workloads likely to remain active;
- installed llama.cpp build, supported devices, and model/context settings;
- free disk for model files, including every shard of split GGUFs.

Do not equate total system RAM with memory available to a model. On shared-memory devices, account for the OS, desktop, and other applications. Check whether llama.cpp can distribute layers across the actual devices; do not assume a vendor name guarantees a backend or full offload.

## 3. Discover and compare current candidates

If the user names a family, search current GGUF variants for it. If they do not, identify suitable families and architectures from up-to-date model cards, architecture/runtime support, and task evidence before looking for quantized artifacts. The `scripts/recommend-models.sh` helper can search Hugging Face and summarize GGUF sizes for a query; use it as a shortlist, not as a quality ranking or exhaustive catalog.

Compare a small, useful set of candidates. For each, report:

- family, architecture, parameter count or active-parameter detail when known, and the exact task variant;
- evidence for required capabilities (tool calling, vision, audio, reasoning/template behavior, languages);
- source repository, license, exact file(s), quantization, download size, and whether the file is split;
- context limit and compatibility evidence for the current llama.cpp version;
- model-specific memory estimate and the uncertainty in that estimate;
- known quality/speed measurements relevant to the user's task, clearly separating benchmarks from vendor claims.

Do not select by parameter count alone. Use current evidence rather than a canned list of family names or a universal threshold such as "X billion parameters for agents." If the request depends on reliable coding, tool calls, vision, or another capability, say what is and is not verified for the exact candidate.

## 4. Estimate fit without overstating precision

For a conventional transformer, the KV cache per token can be approximated from the actual architecture metadata:

```text
KV bytes/token ≈ 2 × layers × KV heads × head dimension × bytes per KV element
KV total ≈ KV bytes/token × context tokens
```

The first factor of 2 accounts for keys and values. Use the model's actual GGUF/HF configuration and selected KV-cache type. Hybrid, recurrent, sliding-window, or otherwise non-standard architectures can need a different estimate; prefer llama.cpp's own load/context reporting when available.

Compare the weight files plus estimated KV cache plus measured/runtime-specific overhead against memory the process can actually use. Include concurrent workloads and the user's desired safety headroom. No fixed extra-GiB allowance is valid for every backend, model, context, or device. If the runtime overhead is unknown, give a range and label the fit as provisional; suggest a smaller candidate or a short trial load only when the user wants to proceed.

The GGUF helper's `--ram` input is the currently available capacity in the selected memory pool for this run, not total installed memory and not an already-reserved model-only budget. Its `--reserve` is additional runtime/workload headroom that the estimate adds to weights and KV before comparing against `--ram`. Derive both from the actual device and workload rather than copying values from another host. Treat partial metadata, an unavailable KV estimate, or unknown multi-GPU placement as unknowns, not as a guaranteed fit.

## 5. Give the user a practical recommendation

Present a primary candidate and one fallback when alternatives meaningfully trade quality, speed, memory, or capability. Explain why each fits the user's intent and constraints. Include the exact source and file set; do not download it unless the user asks.

If a live source cannot be reached, state that current availability could not be confirmed. Continue with user-provided repositories/files or general guidance, clearly separating verified facts from assumptions.
