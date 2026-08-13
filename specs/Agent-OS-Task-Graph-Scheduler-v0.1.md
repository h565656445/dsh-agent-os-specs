---
type: specification
status: implementation-candidate
created: 2026-07-17
updated: 2026-07-17
---

# Agent OS 任务图调度与恢复 v0.1

本规格实现已批准路线图节点 AOS-003：在 AOS-001 运行内核和 AOS-002 Worker 协议之上，提供受控 DAG 调度、并发限制、有界重试、暂停和崩溃恢复。调度器只签发和接收本地结构化票据，不直接调用模型、网络、发布、删除、付款或账号接口。

## 公共 Seam

唯一公共入口：

```powershell
pwsh -File .\10-项目\Hermes-Harness\runner\agent_os_scheduler.ps1 `
  -Action <Initialize|Dispatch|Accept|Pause|Resume|Inspect|Recover>
```

| Action | 必要输入 | 结果 |
| --- | --- | --- |
| `Initialize` | `GraphSpecPath`、`RuntimeRoot` | 验证 DAG 后创建不可变图合同、Ledger、Snapshot |
| `Dispatch` | `GraphPath`、`RuntimeRoot` | 在依赖和并发额度内签发不可变派发票据 |
| `Accept` | `GraphPath`、`ResultPath` | 校验 Worker 回执、幂等键和哈希后提交节点终态 |
| `Pause` | `GraphPath`、`Reason` | 追加人工暂停事件，停止新派发 |
| `Resume` | `GraphPath`、`Reason` | 仅恢复非终态、非对账态任务图 |
| `Inspect` | `GraphPath` | 只读重放可信 Ledger 前缀，不修改运行时 |
| `Recover` | `GraphPath` | 从 Ledger、票据和已落盘结果重建状态 |

`ObservedAt` 只供 `Recover` 判断租约是否到期；它不能使不可逆节点自动重试。

## 机器合同

- `agent_os_graph_spec.schema.json`：输入 DAG、策略和节点引用。
- `agent_os_graph_contract.schema.json`：规范化后的不可变运行合同。
- `agent_os_graph_ticket.schema.json`：派发票据。
- `agent_os_graph_result.schema.json`：调度结果和 Worker 回执绑定。

所有 Schema 均关闭未知字段。规格和结果中的工作引用只保存路径和 SHA-256；调度器不会把工作正文复制进图合同。

## 节点与依赖规则

每个节点必须声明：

- 唯一 `node_id`；
- 已存在的 `depends_on`；
- 每次尝试各自独立的 `worker_input` 路径及 SHA-256，`work_refs` 数量必须等于 `max_attempts`；
- `read_only`、`reversible` 或 `irreversible` 副作用等级；
- 全图唯一 `idempotency_key`；
- `max_attempts`，硬上限为 2。

初始化使用拓扑消除检查循环依赖。未知依赖、重复节点、重复幂等键、工作输入漂移或循环均在创建运行目录前失败。

## 调度规则

1. 只有全部依赖为 `succeeded` 的 `pending` 节点可派发。
2. `running` 节点数不得超过 `max_concurrency`，硬上限为 16。
3. 第 N 次派发只使用 `work_refs[N-1]`；派发前重新计算 Worker 输入哈希，不得复用已经终态的 Worker 作业。不可逆节点还要重新计算独立批准文件哈希。
4. 调度器先以 `CreateNew` 落盘票据，再把 `node_dispatched` 写入 Ledger。
5. 同一节点的票据文件名绑定尝试次数，不能覆盖已有票据。
6. 失败只在剩余预算内回到 `pending`；第二次失败进入 `failed`，按策略暂停全图。

## 恢复与幂等

Ledger 是运行状态事实源，Snapshot 只是可重建缓存。每个 Ledger 事件包含序号、前序哈希和事件哈希；合同漂移、断链或事件篡改都会关闭派发。

| 崩溃窗口 | Recover 行为 |
| --- | --- |
| 合同已写、Snapshot 缺失 | 从可信 Ledger 重建 Snapshot |
| 票据已写、派发事件未写 | 可逆节点补记同一票据；不可逆节点进入 `reconciliation_required` |
| 派发事件已写、结果未到、租约过期 | 可逆节点在预算内重试；不可逆节点进入人工对账 |
| 结果已写、终态事件未写 | 校验结果、Worker 回执和哈希后补记终态，不重新执行 |
| 终态事件已写、Snapshot 落后 | 从 Ledger 重建，不重复提交结果 |
| Ledger 损坏 | 只暴露可信前缀，禁止任何变更和派发 |

不可逆节点必须附带单独的 `approval_ref`。任何“是否已经发生”不确定的不可逆派发都不会自动重试，只能人工对账。这是“中断后不会重复不可逆动作”的实现保证。

## 安全规则

- `RuntimeRoot` 与图目录不得是 Junction、符号链接或其他重解析点。
- 图目录必须是指定 `RuntimeRoot` 的直接子目录。
- 工作输入、批准文件、票据、结果和 Worker 回执在使用前复算 SHA-256；Worker 回执必须通过 AOS-002 Schema，绑定本次输入哈希，并复算其结果和全部证据。
- Worker 结果和回执字节执行疑似密钥扫描；命中后不复制、不写 Ledger。
- 单图变更使用独占锁；并发写入失败关闭。
- `Inspect` 永远只读；`Recover` 只修复本地图账本和缓存。

## 状态

节点状态：`pending / running / succeeded / failed / reconciliation_required`。

图状态：`ready / running / paused / succeeded / failed / reconciliation_required`。

`succeeded`、`failed` 和 `reconciliation_required` 不允许通过普通 `Resume` 越过。人工对账后的专用解除协议不属于 v0.1，不能用修改 Snapshot 代替。

## AOS 边界

AOS-003 交付调度器和恢复/幂等策略，不等于五个业务项目已经全部自动接入。AOS-004 的可观测性、成本与质量建设尚未启动。AOS-003 实现验证后必须停在 `waiting_for_approval`，只有用户明确验收才能进入 `completed`。

导航：[[10-项目/Hermes-Harness/README|Hermes Harness]] · [[10-项目/Hermes-Harness/specs/Agent-OS-Worker-Protocol-v0.1|AOS-002 Worker 协议]]
