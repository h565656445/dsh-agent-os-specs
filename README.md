# dsh-agent-os-specs

<!-- DeepSeek Harness 衍生声明 -->
> **DeepSeek Harness 个人适配声明（Personal Adaptation Notice）**
>
> 本项目是 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 的**个人适配产物（personal adaptation）**，**并非 DeepSeek Harness 官方文件（not an official DeepSeek Harness file）**，随附功能、使用说明与个人产物（bundled with features, documentation, and personal artifacts），可与 DeepSeek Harness 搭配使用，也可独立使用。
>
> This project is a **personal adaptation** for DeepSeek Harness, and is **NOT an official DeepSeek Harness file**, bundled with features, documentation, and personal artifacts. It can be used alongside DeepSeek Harness or standalone.

**作者 / Author**: [h565656445](https://github.com/h565656445)

**合作 / Collaboration**: 如有项目可以一起合作，欢迎联系。微信：`wohaishihenshuaide`。If you have projects, let's collaborate. WeChat: `wohaishihenshuaide`.


---

## 用途 / What this is for

Agent OS 五份规格文档合集：运行内核、规划 Loop、Worker 协议、任务图调度、可观测性，是理解 Agent OS 设计的权威规格。

The five Agent OS specification documents: runtime kernel, planning loop, worker protocol, scheduler, observability.

---
## Agent OS Specifications / Agent OS 规格文档

本仓库收录 Agent OS 路线图的规格文档合集：运行内核（AOS-001）、规划 Loop、Worker 协议（AOS-002）、任务图调度与恢复（AOS-003）、可观测性/成本/质量门禁（AOS-004）。这些规格把「走向 Agent OS」的路线图节点编译为可校验的任务契约与验收映射，是运行内核、调度器、Worker 协议与观测运行时实现的事实依据。本仓库为纯文档仓库，不包含任何源码。

This repository collects the Agent OS roadmap specifications: Runtime Kernel (AOS-001), Planning Loop, Worker Protocol (AOS-002), Task-Graph Scheduler & Recovery (AOS-003), and Observability / Cost / Quality gates (AOS-004). They compile the "toward Agent OS" roadmap nodes into verifiable contracts and acceptance mappings — the factual basis for the kernel, scheduler, worker protocol and observability runtimes. Pure documentation; no source code is included.

## Features / 功能

- **运行内核 v0.1**：任务生命周期、哈希链 Ledger、崩溃恢复、批准与执行授权分离（Initialize / Transition / Inspect / Recover）
  Runtime Kernel v0.1: task lifecycle, hash-chained Ledger, crash recovery, separated approval vs execution authorization
- **规划 Loop v0.1**：八个建设任务依赖图、两轮校验 + 一次修复、人工批准路线
  Planning Loop v0.1: 8-node dependency graph, verify-twice / fix-once, human-approved roadmap
- **Worker 协议 v0.1**：最小上下文、能力与权限适配器、结构化结果与可复算证据回传
  Worker Protocol v0.1: minimal context, capability/permission adapters, structured results with verifiable evidence
- **任务图调度与恢复 v0.1**：DAG 调度、并发上限、有界重试、崩溃窗口恢复与不可逆动作对账
  Task-Graph Scheduler v0.1: DAG dispatch, concurrency caps, bounded retries, crash-window recovery, irreversible-action reconciliation
- **可观测性、成本与质量 v0.1**：统一五维指标、预算预留、fail-closed 质量门与哈希链审计
  Observability, Cost & Quality v0.1: unified five-dimension metrics, budget reservation, fail-closed quality gates, hash-chained audit

## What's inside / 目录结构

```
dsh-agent-os-specs/
├── README.md              # 双语说明 + DSH 衍生声明
├── LICENSE                # MIT
├── specs/                 # Agent OS 规格文档（5 份，纯文档）
│   ├── Agent-OS-Runtime-Kernel-v0.1.md
│   ├── Agent-OS-Planning-Loop-v0.1.md
│   ├── Agent-OS-Task-Graph-Scheduler-v0.1.md
│   ├── Agent-OS-Worker-Protocol-v0.1.md
│   └── Agent-OS-Observability-Cost-Quality-v0.1.md
└── .dsh/                  # DeepSeek Harness 衍生包
    ├── preset.yml
    ├── agent.cordis.yml
    ├── README.md
    └── skills/dsh-agent-os-specs/SKILL.md
```

## Quick start / 快速开始

```powershell
# 1. 浏览规格清单
$repo = "E:\path\to\dsh-agent-os-specs"
Get-ChildItem (Join-Path $repo "specs") -Filter *.md | Select-Object Name

# 2. 阅读某份规格
Get-Content (Join-Path $repo "specs\Agent-OS-Runtime-Kernel-v0.1.md")

# 3. 安装 DSH 预设（可选）
$dst = Join-Path $env:DSH_HOME ".agent-presets\agent-os-specs"
Copy-Item -Recurse -Force (Join-Path $repo ".dsh") $dst
```

## DeepSeek Harness 衍生 / DSH Derivative

本项目附带 DeepSeek Harness 衍生包，位于 `.dsh/` 目录：

- `preset.yml` — Agent 预设元数据
- `agent.cordis.yml` — Cordis 组装（基于 standard 预设，persona 已定制）
- `skills/dsh-agent-os-specs/SKILL.md` — 项目专属技能（skill）

安装与接入方式见 [`.dsh/README.md`](.dsh/README.md)（双语）。


## License / 许可证

[MIT](LICENSE)

---

## 相关项目 / Related Projects

> 这是 DeepSeek Harness 个人适配系列（共 40 个仓库）的完整导航。 / This is the complete navigation for the DeepSeek Harness personal-adaptation series (40 repos).

### Agent OS 内核 / Kernel

[`dsh-agent-os-runtime`](https://github.com/h565656445/dsh-agent-os-runtime) · [`dsh-agent-os-planning`](https://github.com/h565656445/dsh-agent-os-planning) · [`dsh-agent-os-scheduler`](https://github.com/h565656445/dsh-agent-os-scheduler) · [`dsh-agent-os-worker-protocol`](https://github.com/h565656445/dsh-agent-os-worker-protocol) · [`dsh-agent-os-observability`](https://github.com/h565656445/dsh-agent-os-observability) · **`dsh-agent-os-specs`（本仓库 / this repo）**

### Harness 基础设施 / Infrastructure

[`dsh-harness-core`](https://github.com/h565656445/dsh-harness-core) · [`dsh-graph-entry`](https://github.com/h565656445/dsh-graph-entry) · [`dsh-async-job`](https://github.com/h565656445/dsh-async-job) · [`dsh-file-identity`](https://github.com/h565656445/dsh-file-identity) · [`dsh-json-projection`](https://github.com/h565656445/dsh-json-projection) · [`dsh-manual-approval`](https://github.com/h565656445/dsh-manual-approval) · [`dsh-observation-writer`](https://github.com/h565656445/dsh-observation-writer) · [`dsh-provider-control`](https://github.com/h565656445/dsh-provider-control) · [`dsh-schema-negotiator`](https://github.com/h565656445/dsh-schema-negotiator) · [`dsh-schema-registry`](https://github.com/h565656445/dsh-schema-registry) · [`dsh-upgrade-governance`](https://github.com/h565656445/dsh-upgrade-governance) · [`dsh-task-contract`](https://github.com/h565656445/dsh-task-contract) · [`dsh-quality-gates`](https://github.com/h565656445/dsh-quality-gates) · [`dsh-worker-tests`](https://github.com/h565656445/dsh-worker-tests)

### Worker 与管线 / Workers & Pipelines

[`dsh-codex-worker`](https://github.com/h565656445/dsh-codex-worker) · [`dsh-novel-chapter-trial`](https://github.com/h565656445/dsh-novel-chapter-trial) · [`dsh-novel-video-pipeline`](https://github.com/h565656445/dsh-novel-video-pipeline) · [`dsh-portfolio-routing`](https://github.com/h565656445/dsh-portfolio-routing) · [`dsh-meta-agents-bridge`](https://github.com/h565656445/dsh-meta-agents-bridge)

### 规格与文档 / Specs & Docs

[`dsh-harness-specs`](https://github.com/h565656445/dsh-harness-specs) · [`dsh-novel-specs`](https://github.com/h565656445/dsh-novel-specs) · [`dsh-architecture-guide`](https://github.com/h565656445/dsh-architecture-guide) · [`dsh-powershell-patterns`](https://github.com/h565656445/dsh-powershell-patterns) · [`dsh-json-schema-driven-dev`](https://github.com/h565656445/dsh-json-schema-driven-dev) · [`dsh-llm-agent-harness-guide`](https://github.com/h565656445/dsh-llm-agent-harness-guide)

### 适配器 / Adapters

[`dsh-short-story-engine`](https://github.com/h565656445/dsh-short-story-engine) · [`dsh-tutorial-video-state-machine`](https://github.com/h565656445/dsh-tutorial-video-state-machine) · [`dsh-governance-kernel`](https://github.com/h565656445/dsh-governance-kernel) · [`dsh-sports-pipeline`](https://github.com/h565656445/dsh-sports-pipeline) · [`dsh-motion-grammar`](https://github.com/h565656445/dsh-motion-grammar)

### DSH 总集成 / Integration

[`dsh-integration`](https://github.com/h565656445/dsh-integration) · [`dsh-presets-pack`](https://github.com/h565656445/dsh-presets-pack) · [`dsh-skills-pack`](https://github.com/h565656445/dsh-skills-pack) · [`dsh-starter-kit`](https://github.com/h565656445/dsh-starter-kit)

