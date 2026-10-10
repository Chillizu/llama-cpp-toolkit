# Build your own manager

A personal, llmctl-style model manager is a wrapper over `llama-server` that resolves models, assembles launch options, manages server lifecycle, and can connect an OpenAI-compatible client. `scripts/llmctl-template.sh` is the single-model starter implementation. This guide also draws on the richer local `llmctl` in use with llama-ranch; those observed capabilities are reference points and are not all implemented by the template.

## Design

State kept in `$STATE_DIR` (per-process files, not a database):

- `chat.pid` — running server PID.
- `chat.model` — resolved model name.
- `chat.args` — the exact llama-server args in use (needed to identify which profile matches, and to rebuild for `pi`).
- `chat.type` — `single` in `llmctl-template.sh`. A router is a separate server mode in the local `llmctl` and needs its own lifecycle and readiness behavior.

One chat server at a time: `start` stops the previous one first. This avoids GPU/RAM contention from two loaded models.

## Subcommands

| command | behavior |
|---|---|
| `list` | enumerate `.gguf` models (recursive scan, numbered) + saved profiles per model |
| `start <name\|#\|name@variant> [args…]` | resolve query → model, load profile (or variant), stop prior server, launch, health-wait |
| `stop` | terminate the running chat server |
| `status` | running model, key params, matching profile, endpoint, health probe |
| `profile <name> [variant] [args…]` | set/remove a profile (no args = remove) |
| `profile --show [name]` | view profile(s) |
| `client [<model|#> [variant]] [-- cmd…]` | ensure server + health, then exec `cmd` with `OPENAI_BASE_URL`/`OPENAI_API_KEY` set (no `cmd` → print the export lines) |

## Profiles

- Location: `<PROFILE_DIR>/<model>` for the default profile, `<PROFILE_DIR>/<model>@<variant>` for a named variant.
- Content: a **single line** of llama-server args.
- **Trailing newline is mandatory** — a missing `\n` makes `read -ra` fail silently under `set -e`. Write with `printf '%s\n'`, verify with `tail -c1 | od -An -c`.
- `start` with no extra args uses the saved default/variant profile; extra args override/append for that launch.

## UX safeguards worth keeping

- **`model@variant` sugar**: `start qwen longctx` and `start qwen@longctx` are equivalent; translate the latter to the two-arg form.
- **Port-busy guard**: before launching, check the target port; if an unrelated process owns it, abort and print the owner PID + kill hint. This prevents the "silent wrong server" trap (your request hitting an old instance on the same port).
- **Early-death check**: 2s after launch, if the PID died, dump the log tail and exit 1 (catches bad flags / port races instantly).
- **Built-in health wait**: poll `/health` (print `ok` or `loading…`); cold model load can take a while — `client` waits up to a generous timeout.
- **Priority boost**: `renice -10` + `oom_score_adj -1000` (needs privileged access), optional.
- **Status profile line**: `status` shows which saved profile (or `custom`) the running args match — important when a profile was overridden at launch.

## Feature recommendations from the local llmctl

The local `llmctl` adds capabilities beyond this starter: XDG-aware paths and a user config file; stable model aliases; an explicit Pi adapter with workspace/session controls and provider synchronization; and a separate llama-server router mode with on-demand model loading. Treat these as evidence of useful patterns, then select features from the user’s machine, client, and workflow rather than copying this particular setup wholesale.

Recommended design directions:

- **Configuration:** keep machine-specific paths and tuning outside the script, with documented precedence between config-file values and environment overrides. Explain that a sourced shell config executes as code; use a constrained data format if that trust model is undesirable. Discover options from the installed server/backend and the user’s actual needs instead of presenting a fixed universal list.
- **Client integration:** make integrations optional adapters. For Pi, provide a deliberate command that selects a stable model alias, connects to the managed endpoint, and lets the user choose an isolated workspace/session or an existing project workspace. Keep generated provider data scoped, preserve unrelated settings, and make changes to a client’s default provider/model explicit and reversible.
- **Router mode:** model it as a distinct mode, not a variation of single-model profiles. Derive or ask for a safe model-load limit based on available memory and intended use; do not promote the local two-model cap as a universal default. Give router mode its own state, logs, readiness checks, and clear explanation of which single-model profile settings do not apply.
- **Profiles and overrides:** support a default plus user-named variants when repeated launch configurations are useful. Show which profile is active. Define whether command-line options extend or replace profile values and surface conflicts, so a launch cannot silently discard a user’s saved tuning.
- **Diagnostics:** distinguish process startup, endpoint health, and model readiness. Check early process exit and report a useful log tail; expose the active endpoint, mode, model/profile, and health state in `status`. Apply these checks to both single-model and router modes.
- **Hardware tuning:** suggest candidate settings from detected hardware and the selected backend, then explain tradeoffs and let the user opt in. Do not bake this host’s CPU lists, memory limits, vendor flags, or benchmark results into a general skill.

These are optional capabilities, not a required checklist. Before recommending one, ask whether the user wants that capability when it changes their workflow (for example, Pi integration or router mode), inspect the local command’s help/config and the installed backend for supported options, and tailor the explanation to that evidence. Avoid fixed option glossaries: generate concise, contextual explanations for the options actually available on the user’s system.

## Adapting the template

1. **Paths**: set `MODEL_DIR`, `STATE_DIR`, `CONFIG_DIR`, `LOG_DIR` for your layout.
2. **Backend flags**: run `scripts/detect-device.sh`, then set `BACKEND_FLAGS` per the mapping in `references/device-detection.md` (e.g. `-ngl -1 --no-mmap` for Vulkan integrated; `-ngl -1` for CUDA/Metal).
3. **CPU pinning**: set `PIN_CPUS` to your P-core list (from `lscpu -p` / CPU topology sysfs) so decode doesn't run on efficiency cores.
4. **Defaults**: adjust `DEFAULT_PORT` and the default sampler/context set in `references/tuning.md` for your workload.
5. **Verify**: `list`, `start <model>`, `status` (health ok), `profile <model> -c 8192`, then `stop`. Wire `client` last, confirming `OPENAI_BASE_URL` reaches the local endpoint.

Rename the script (`llmctl-template.sh` → `llmctl` or your own name) and drop it on your `PATH`.
