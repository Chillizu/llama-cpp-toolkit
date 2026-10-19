# llama-cpp-toolkit

> 本地 LLM 工具包，聚焦 llama.cpp 与 GGUF：先了解你的用途，再审视本机条件、比较当前可获取的模型，并按实测逐步跑通与调优。
>
> A local LLM toolkit focused on llama.cpp and GGUF: start from your task and hardware, compare current model options, then set up and tune from measured results.

Formerly **llama-ranch**.

[中文](#中文) · [English](#english)

## 中文

llama-cpp-toolkit 是一个以 Agent Skill 为主、附带小型脚本和参考资料的本地推理工具包。它根据用户目标和当前设备情况，动态引导模型选型、来源核对、运行环境准备和推理调优。

### 安装

```bash
npx skills add Chillizu/llama-cpp-toolkit
```

### 首次使用

在支持 Agent Skills 的 Agent 中，描述任务和偏好：

> 我想在本地做代码协作，优先保护隐私，也希望模型能调用工具。先检查这台机器的硬件和已有推理环境，再比较适合的模型家族、架构、规模和可获取的 GGUF 文件。解释推荐依据、来源和不确定性；现在只给建议，不要下载、安装或启动。

选定具体模型来源和文件后，如果你明确要求它代为准备和启动，它应先复用已有运行时，并根据检测到的系统、后端和模型生成步骤。若模型来源、持久配置或服务范围仍有实质歧义，再先与你确认。

### 能做什么

- **审视本机**：识别操作系统、CPU、可用内存、GPU 与现有推理环境；将设备检测脚本作为线索，并用实际运行时信息校正。
- **按任务选模型**：根据用途、质量/速度偏好、上下文、工具或模态需求，动态比较当前模型家族、架构、规模、量化和 GGUF 来源，不使用固定的“最佳模型”清单。
- **估算并核对适配度**：把权重、模型架构对应的 KV cache、上下文和后端开销纳入估算；区分独立显存与共享内存，并说明不确定性。
- **跑通与调优**：按现有环境准备 llama.cpp/llama-server；先完成代表性任务的基线，再根据实际速度、内存和效果逐项调优。
- **接入客户端**：按需连接本地 OpenAI 兼容端点，并验证客户端和模型是否具备所需能力。
- **可选管理器**：`scripts/llmctl-template.sh` 提供个人管理器起点；它需要适配，不是首次使用的前置条件。

首次模型跑通后，如果用户会反复切换模型、配置启动方案或连接客户端，再询问是否需要 `llmctl` 风格的管理器；只需建议时就不生成管理配置。

### 脚本是辅助工具

```text
scripts/detect-device.sh       设备信息摘要；启发式结果需用本机运行时校验
scripts/recommend-models.sh    实时 Hugging Face GGUF 文件搜索与内存估算
scripts/llmctl-template.sh     个人模型进程管理器模板
references/*.md               按需读取的检测、选型、安装、调优和排障资料
```

需要单独查询时，模型搜索助手可接收具体候选家族、可用于模型的内存预算和目标上下文：

```bash
scripts/recommend-models.sh "<candidate-family>" --ram <available-memory-gib> --ctx <tokens> [--reserve <runtime-headroom-gib>]
```

`--ram` 是所选内存池当前可供本次运行使用的容量，不是机器标称总内存；`--reserve` 是还要为运行时和其他占用留出的余量，会在估算中另行扣除。两者应来自当前设备和工作负载的实际情况。未提供上下文或运行时余量时，助手不会给出确定的 fit 结论。Skill 会根据系统、用户目标和可用工具决定是否调用脚本；脚本结果不取代模型能力、来源、运行时兼容性和硬件适配的综合判断。

### 范围与要求

- 当前重点是 `llama.cpp` 与 GGUF；其他本地运行时可以讨论，但会标明未经本项目工具验证的部分。
- 不包含 `llama.cpp` 可执行文件或模型权重，也不默认下载、安装或启动服务。
- 模型来源查询依赖网络；模型搜索脚本使用 Hugging Face。网络受限时应说明限制，并根据用户提供的文件或来源资料继续。
- 脚本需要 bash、`curl` 和 `python3`；`jq` 可选。硬件检测依赖各操作系统提供的诊断工具，平台覆盖并不完全相同。
- 仅指导推理期运行和调优，不训练或微调权重。

---

## English

llama-cpp-toolkit is an Agent Skill with small helper scripts and on-demand references. It adapts model discovery, source checks, setup, and inference tuning to the user's goal and the machine's current state.

### Install

```bash
npx skills add Chillizu/llama-cpp-toolkit
```

### First use

In an agent that supports Agent Skills, describe your task and preferences:

> I want a local coding assistant, with privacy as a priority and reliable tool use. Inspect this machine's hardware and existing inference setup, then compare suitable model families, architectures, size tiers, and available GGUF files. Explain the evidence, sources, and uncertainty. Give recommendations only; do not download, install, or start anything yet.

After choosing an exact source and file, explicitly ask it to perform setup and launch if you want those actions. It should reuse an existing runtime and tailor steps to the detected OS, backend, and model. It should ask first only when the model source, persistent changes, or service scope is materially ambiguous.

### Capabilities

- **Inspect the machine:** identify the OS, CPU, available memory, GPU, and existing inference setup. Treat detector output as a clue and cross-check it with the actual runtime.
- **Choose for the task:** dynamically compare current model families, architectures, size tiers, quantizations, and GGUF sources against the task, quality/speed preferences, context, tool use, and modality needs. There is no fixed “best model” list.
- **Estimate and verify fit:** account for weights, architecture-specific KV cache, context, and backend overhead; distinguish dedicated VRAM from shared memory and state uncertainty.
- **Run and tune:** prepare llama.cpp/llama-server for the existing environment, establish a representative baseline, then tune against measured speed, memory, and task quality.
- **Connect clients:** wire a local OpenAI-compatible endpoint when requested and verify the client and model support the needed capability.
- **Optional manager:** `scripts/llmctl-template.sh` is a starting point for personal process management. It needs adaptation and is not required for first use.

After the first model works, if the user expects to switch models, reuse launch profiles, or connect clients repeatedly, ask whether they want an `llmctl`-style manager. If they only need recommendations, do not generate manager configuration.

### Helper scripts

```text
scripts/detect-device.sh       hardware summary; heuristic results need runtime verification
scripts/recommend-models.sh    live Hugging Face GGUF file search and memory estimate
scripts/llmctl-template.sh     personal model-manager template
references/*.md               on-demand detection, selection, setup, tuning, and troubleshooting guides
```

For a targeted lookup, the model-search helper accepts a candidate family, the memory budget available to the model, and the intended context:

```bash
scripts/recommend-models.sh "<candidate-family>" --ram <available-memory-gib> --ctx <tokens> [--reserve <runtime-headroom-gib>]
```

`--ram` is the currently available capacity of the selected memory pool for this run, not the machine's advertised total. `--reserve` is additional runtime and workload headroom, which the estimate subtracts separately. Base both values on current device and workload evidence. Without context or runtime headroom, the helper will not report a definite fit. The skill decides whether to use a helper based on the OS, user goal, and available tools. Script output does not replace checking model capability, source, runtime compatibility, and hardware fit.

### Scope and requirements

- Current focus: `llama.cpp` and GGUF. Other local runtimes can be discussed, with unverified support clearly identified.
- The repository does not bundle `llama.cpp` binaries or model weights, and does not download, install, or start services by default.
- Model-source discovery needs network access; the model-search helper uses Hugging Face. If access is blocked, state the limitation and continue from user-provided files or source details.
- The shell helpers need bash, `curl`, and `python3`; `jq` is optional. Hardware inspection depends on OS-native diagnostic tools, so platform coverage varies.
- Covers inference and runtime tuning only; it does not train or fine-tune weights.
