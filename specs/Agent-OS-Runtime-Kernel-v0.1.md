---
type: spec
status: implemented
created: 2026-07-17
updated: 2026-07-17
---

# Agent OS Runtime Kernel v0.1

## 目标

AOS-001 把已批准的 Agent OS 路线图节点转换为可运行、可核验、可恢复的任务实例。内核只管理运行合同、生命周期、Ledger 和 Snapshot；它不自动创建后续任务，不自动完成任务，也不修改业务项目。

## 权威链

运行实例按以下顺序建立信任：

1. `plan.json` 必须仍处于 `approved`，并通过规划模块实时复核。
2. 本地执行授权文件必须绑定计划 ID、计划路径、批准哈希和任务键。
3. Runner 调用时必须提供执行授权文件的预期 SHA256；文件有任何漂移都拒绝初始化，任务键命中 `start_<task>` 排除项时也必须拒绝。
4. `contract.json` 从已批准的任务节点生成，创建后不得修改。
5. `ledger.jsonl` 是状态恢复事实源；`snapshot.json` 只是可重建缓存。

规划批准与执行授权彼此分离。AOS-001 的授权范围固定为 `implement + verify`，明确排除“未经用户确认完成任务”和“启动 AOS-002”。

## 公开接口

唯一入口为 `runner/agent_os_runtime.ps1`：

- `Initialize`：验证父计划和执行授权，创建合同、首条 Ledger 事件和 Snapshot。
- `Transition`：执行一个合法状态迁移，并把结果追加到哈希链。
- `Inspect`：只从合同和 Ledger 计算当前可信状态，不修复文件。
- `Recover`：验证完整哈希链；链完整时用 Ledger 重建缺失或过期的 Snapshot。

Runner 不暴露模块内部的序列化、哈希或恢复函数。

## 运行目录

```text
runtime/agent_os_tasks/<task_id>/
├── contract.json
├── ledger.jsonl
└── snapshot.json
```

`task_id` 同时是目录名。任何运行操作都必须位于调用方指定的 `RuntimeRoot` 内；RuntimeRoot 的任一已存在上级路径、任务目录和三个运行文件均不得通过 junction、symlink 或其他 reparse point 跳出边界。

## 合同

`contract.json` 必须通过 `schemas/agent_os_runtime_task.schema.json`。核心字段包括：

- AOS 任务键、标题、目标、能力、交付物和验收条件；
- 父计划路径、计划 ID、任务键和批准哈希；
- 执行授权路径与授权文件哈希；
- 最多两次尝试、一次修正；
- `automatic_dispatch = false`；
- `completion_gate = human_approval_required`。

合同哈希写入每一条 Ledger 事件。合同漂移会使 Ledger 校验失败并关闭派发。

## 生命周期

```text
created -> ready
ready -> running | blocked | failed | cancelled
running -> verifying | repairing | blocked | failed | cancelled
verifying -> waiting_for_approval | repairing | failed
repairing -> running | failed
blocked -> ready | failed | cancelled
waiting_for_approval -> completed | repairing | failed
```

`completed`、`failed`、`cancelled` 是终态。终态之后不接受任何迁移或重新派发。

从 `waiting_for_approval` 进入 `completed` 时，Runner 必须收到显式 `ApproveCompletion`。该开关只对本次完成迁移有效，不是长期授权。

每次进入 `running` 计为一次执行，每次进入 `repairing` 计为一次修正。用量只从可信 Ledger 计算；第三次执行或第二次修正请求不会进入目标状态，而是记录失败事件并进入 `failed`。

## Ledger 与恢复

每条事件记录：

- 单调递增序号；
- 前态与后态；
- 合同 SHA256；
- 前一事件 SHA256；
- 当前事件 SHA256；
- 原因、证据路径和完成批准标记。

恢复从第一条事件开始验证结构、序号、合同哈希、前向哈希、事件哈希和生命周期。遇到损坏立即停止：

- 返回最后一个可信前缀状态；
- 标记 `ledger_integrity = broken`；
- 强制 `dispatch_allowed = false`；
- 不用不可信状态覆盖 Snapshot。

链完整但 Snapshot 缺失或过期时，`Recover` 用 Ledger 重新生成 Snapshot。`dispatch_allowed`、任务键、状态、序号或哈希任一不一致都视为过期快照。

迁移原因、证据路径和所有写入合同的文本在持久化前执行凭据模式检查；命中时不创建任务或追加 Ledger。

## AOS-001 验收映射

| 路线图验收条件 | 运行内核证据 |
|---|---|
| 任一任务可从账本恢复最后可信状态 | `Recover` 从完整链重建 Snapshot；损坏时只暴露可信前缀 |
| 终态不会被自动重派 | 终态 `dispatch_allowed = false`，并拒绝后续 `Transition` |

## 非目标

v0.1 不实现多 Worker 调度、跨进程锁、自动补偿、长期记忆晋级或递归自我修改。这些能力必须由后续经批准的 AOS 节点逐项引入。
