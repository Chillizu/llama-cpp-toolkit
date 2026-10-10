# llama-ranch

> 设备无关的本地 GGUF 大模型 Agent Skill —— 在任何 GPU 上跑、选、调本地模型。
> Device-agnostic Agent Skill for running, recommending, and tuning local GGUF LLMs on any GPU.

[English](#english) · [中文](#中文)

## 中文

**llama-ranch** 帮你走完"从零到跑起来并调好"的全流程：检测本机硬件 → 选一个适配显存/内存的模型与量化 → 推理期调参（量化级别 / 上下文长度 / 线程数 / 采样器 / KV cache 类型）→ 启动 llama-server → 可选接入 agentic 客户端（pi / OpenCode）到本地 OpenAI 兼容端点。全部经验去厂商化、无硬编码数字，按你的机器实测为准。

### 安装

```bash
npx skills add Chillizu/llama-ranch
```

### 首次使用

安装后，在支持 Agent Skills 的目标 Agent 中这样开始：

> 用 llama-ranch 检查本机硬件，并推荐一个能放进可用显存/内存的 GGUF 模型。先只给建议，不下载、不安装、不启动服务。

选定模型后，可以继续要求它按检测到的后端带你准备运行环境；也可以提供已有模型和 `llama-server` 路径，让它帮你拟启动参数、接入客户端。

> 我已经有模型文件和 llama-server。根据本机硬件给出启动命令，并说明每个参数；先不要执行。

llama-ranch 是指导与脚本集合，不包含 `llama.cpp`、模型权重或已配置好的完整管理器。设备检测和模型搜索脚本可直接运行；`llmctl-template.sh` 是需要按本机路径和工作流调整的模板。模型搜索需要访问 Hugging Face。

### 功能

- **检测设备** → `scripts/detect-device.sh`：OS、GPU 厂商/型号/显存（独显 vs 共享堆）、RAM、CPU 核数与 P/E 分层、建议后端（vulkan / cuda / rocm / metal / cpu），输出 JSON。多显卡时优先独显。
- **选模型** → `scripts/recommend-models.sh "<名字>" --ram <gib> [--ctx <tokens>]`：实时 HuggingFace GGUF 搜索，含各量化体积与精确 fp16 KV cache 预算 → fits 提示。无硬编码模型表；分片 GGUF 自动合并求和；网络屏蔽 HF 时设 `HF_PROXY`。
- **调优** → `references/tuning.md`：容量公式、KV cache 数学（fp16 / q4_0 / q8_0 取舍）、`-c` 上下文档位、`-t` 线程扫描、采样器设置、decode 天花板、衰减行为。
- **运行** → `llama-server -m model.gguf <flags>`，或用 `scripts/llmctl-template.sh`（参数化 `list/start/stop/status/profile/client`）搭个人管理器。
- **排障** → `references/stability.md`：通用 bug/修复矩阵（reasoning 模板吞输出、DeviceLost 竞态、q8_0 KV 毒化、工具参数退化、profile 缺尾换行）。
- **Agentic** → `references/agentic.md`：任意 OpenAI 兼容客户端接本地端点（`OPENAI_BASE_URL` 注入）、工具调用验证、agent bench 方法论。
- **从零起步** → `references/bootstrap.md`：装/编 llama.cpp（按检测后端选 cmake flag）、验证 `--list-devices`、下载模型。

### 目录结构

```
SKILL.md                          frontmatter(name/description) + 决策流
scripts/detect-device.sh          一键设备能力报告 → JSON
scripts/recommend-models.sh       实时 HuggingFace GGUF 搜索 + fits 提示
scripts/llmctl-template.sh        参数化个人模型管理器
references/*.md                   按需加载的细节文档（bootstrap, detection, selection, tuning, stability, agentic, manager）
```

### 说明

- 仅推理期调参，不含训练。
- 经验已去厂商化：不硬编码设备专属吞吐/内存数字，按你的硬件实测。
- 客户端无关：`client` 子命令注入 `OPENAI_BASE_URL`/`OPENAI_API_KEY`，任何 OpenAI 兼容客户端（CLI、聊天 UI、agent）均可接入。
- 依赖：bash（兼容 3.2，推荐 4+）、curl、python3；jq 可选（detect-device 缺失时自动回退 python3）。

---

## English

**llama-ranch** takes you from zero to a tuned local model: detect the machine → pick a model + quantization that fits its VRAM/RAM → tune inference-time settings (quantization level, context length, threads, samplers, KV-cache type) → run a llama-server → optionally wire an agentic client (pi / OpenCode) to the local OpenAI-compatible endpoint. Empirical insight is de-vendored; measure on your hardware.

### Install

```bash
npx skills add Chillizu/llama-ranch
```

### First use

After installation, start in an agent that supports Agent Skills:

> Use llama-ranch to inspect this machine and recommend a GGUF model that fits available VRAM/RAM. Give me advice only; do not download, install, or start anything.

Once you choose a model, ask it to guide setup for the detected backend. If you already have a model and `llama-server`, give it their paths and ask for a launch command and client wiring.

> I already have a model file and llama-server. Give me a launch command for this machine and explain each option; do not run it yet.

llama-ranch is a guide and script collection. It does not bundle `llama.cpp`, model weights, or a fully configured manager. Device detection and model search scripts can be run directly; `llmctl-template.sh` must be adapted to your paths and workflow. Model search requires access to Hugging Face.

### What it does

- **Detect** → `scripts/detect-device.sh`: OS, GPU vendor/model/memory (dedicated vs shared heap), RAM, CPU cores incl. P/E split, suggested backend (vulkan / cuda / rocm / metal / cpu). Emits JSON. Prefers the discrete GPU on multi-adapter machines.
- **Pick** → `scripts/recommend-models.sh "<name>" --ram <gib> [--ctx <tokens>]`: live HuggingFace GGUF search with per-quant file sizes and exact fp16 KV-cache budget → fits hints. No hardcoded model tables; split GGUFs are merged and summed. Set `HF_PROXY` if your network blocks HF directly.
- **Tune** → `references/tuning.md`: capacity formula, KV-cache math (fp16 / q4_0 / q8_0 tradeoffs), `-c` context tiers, `-t` thread sweep, sampler setup, decode ceiling, decay behavior.
- **Run** → `llama-server -m model.gguf <flags>`, or build a personal manager from `scripts/llmctl-template.sh` (parameterized `list/start/stop/status/profile/client`).
- **Fix** → `references/stability.md`: generic bug/fix matrix (reasoning-preserving template swallow, DeviceLost races, q8_0 KV poisoning, tool-arg degeneration, trailing-newline profile bug).
- **Agentic** → `references/agentic.md`: wire any OpenAI-compatible client to the local endpoint (`OPENAI_BASE_URL` injection), tool-call verification, agent bench methodology.
- **Bootstrap** → `references/bootstrap.md`: build/install llama.cpp with the detected backend, verify `--list-devices`, download a model.

### Layout

```
SKILL.md                          frontmatter (name/description) + decision flow
scripts/detect-device.sh          one-shot device capability report → JSON
scripts/recommend-models.sh       live HuggingFace GGUF search + fits hints
scripts/llmctl-template.sh        parameterized personal model manager
references/*.md                   on-demand detail (bootstrap, detection, selection, tuning, stability, agentic, manager)
```

### Notes

- Inference-time tuning only — no training.
- Empirical insight is de-vendored: no device-specific throughput/memory numbers are hardcoded; measure on your hardware.
- Client-agnostic: the `client` subcommand injects `OPENAI_BASE_URL`/`OPENAI_API_KEY`, so any OpenAI-compatible client (CLI, chat UI, agent) plugs in.
- Requires bash (3.2-compatible, 4+ recommended), curl, python3; jq optional (detect-device falls back to python3).
