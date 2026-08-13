---
name: dsh-agent-os-specs
description: Agent OS 规格文档的解读与应用专家：运行内核、规划 Loop、任务图调度、Worker 协议、可观测性/成本/质量门禁。 / Expert for the Agent OS specification collection: runtime kernel, planning loop, task-graph scheduler, worker protocol, and observability/cost/quality gates.
---

# Agent OS 规格文档专家 / Agent OS Specification Expert

本技能面向 Agent OS 系列规格（AOS-001 运行内核、规划 Loop、AOS-002 Worker 协议、AOS-003 任务图调度与恢复、AOS-004 可观测性/成本/质量门禁），文档位于仓库 `specs/` 目录，均为纯文档。实施、验证或评审 Agent OS 节点时，以这些规格中的公开接口、不变量、机器合同与验收映射为事实依据。

This skill covers the Agent OS specification set (AOS-001 Runtime Kernel, Planning Loop, AOS-002 Worker Protocol, AOS-003 Task-Graph Scheduler & Recovery, AOS-004 Observability/Cost/Quality gates) in `specs/`. When implementing, verifying or reviewing Agent OS nodes, treat these specs — public interfaces, invariants, machine contracts and acceptance mappings — as the source of truth.

## When to use / 何时使用

实现或评审 Agent OS 节点（AOS-001~AOS-004 及规划 Loop）时；需要设计带哈希链账本、崩溃恢复、权限分离的 Agent 运行时；需要 Worker 最小上下文与结构化证据协议；需要任务图调度、有界重试与不可逆动作对账；需要可观测性指标、预算与质量门禁。

When implementing or reviewing Agent OS nodes; designing agent runtimes with hash-chained ledgers, crash recovery and separated authorization; worker minimal-context and structured-evidence protocols; DAG scheduling with bounded retries and irreversible-action reconciliation; or observability metrics, budgets and quality gates.

## Workflow / 工作流

1. 先读规划 Loop 规格，理解八个建设任务与依赖、批准与执行授权分离。
2. 按需阅读对应节点规格：运行内核（生命周期/Ledger/恢复）→ Worker 协议（适配器/最小上下文/回传）→ 任务图调度（DAG/并发/崩溃窗口）→ 可观测性（指标/预算/质量门）。
3. 用每份规格的「验收映射」核对实现证据；未实现或等待批准的状态不要当作已完成。

1. Start with the Planning Loop spec for the 8-node roadmap and the separation of approval vs execution authorization.
2. Then read the relevant node specs: Runtime Kernel (lifecycle/ledger/recovery) → Worker Protocol (adapters/minimal context/receipts) → Scheduler (DAG/concurrency/crash windows) → Observability (metrics/budget/gates).
3. Use each spec's acceptance mapping to verify implementation evidence; never treat unimplemented or pending-approval items as done.

## References / 参考

- 项目 README: 见仓库根目录
- 作者: h565656445 (GitHub)