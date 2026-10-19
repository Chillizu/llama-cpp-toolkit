# Troubleshooting patterns

Use symptom, logs, model metadata, runtime version, and backend evidence to form a diagnosis. Treat each item below as a hypothesis to verify on the current setup, not a universal mapping from symptom to one flag.

## Reasoning or visible content is missing

- **Symptom**: a response appears blank, or text is only present in a separate reasoning field.
- **Check**: inspect the raw API response, selected model's chat template, reasoning mode, and client handling of content/reasoning fields.
- **Next step**: compare the model's expected template with current llama.cpp options and client behavior. Change one supported option at a time and verify that the user-visible field contains the intended answer.

## Server exits or loses a GPU device during a request

- **Symptom**: process exit, `DeviceLost`, GPU queue submission errors, or failures after a particular prompt/quant/workload.
- **Check**: preserve the full server log and exact command; note backend, driver, llama.cpp build, model architecture/quant, memory use, and whether streaming/tool use/prefill changes the trigger.
- **Next step**: reproduce with a smaller workload and compare a known-supported quant/backend. Apply backend-specific workarounds only when current issue/build evidence supports them. For a confirmed Vulkan submission issue, the user can opt into `GGML_VK_SERIALIZE_SUBMISSIONS=1`; the manager template leaves this unset by default.

## Output becomes repetitive or corrupted

- **Symptom**: healthy server but malformed or repeating output, often after multiple turns.
- **Check**: exact model and template, KV-cache type, context occupancy, backend/build version, and raw response fields.
- **Next step**: return to a known-supported cache type for this model/backend and compare a short single-turn and multi-turn request. Do not assume one cache type is defective across all architectures.

## Tool-call arguments are malformed

- **Symptom**: invalid JSON, repeated characters, missing tool calls, or arguments that do not match the schema.
- **Check**: model capability and template, server tool parser, client schema, sampler options supported by this build, and an actual raw tool-call response.
- **Next step**: simplify the request, confirm parser/architecture compatibility, then change only supported sampler settings and re-run the same tool task. Do not apply a canned sampler chain without verifying the model and runtime.

## Manager exits while loading a profile

- **Symptom**: `llmctl-template.sh start` exits while reading a profile.
- **Check**: inspect the profile file, its quoting, and final newline; shell argument files are not a general-purpose configuration format.
- **Next step**: preserve the required newline and validate options against the installed `llama-server --help`. The template handles a failed `read` defensively, but profile content still needs review.

## Verify a fix

Repeat the request that produced the failure, then check the raw result, server logs, memory, and process state. For agent or multimodal use, verify that exact capability through the intended client. Keep the before/after command and evidence so the change can be reversed if the diagnosis was wrong.
