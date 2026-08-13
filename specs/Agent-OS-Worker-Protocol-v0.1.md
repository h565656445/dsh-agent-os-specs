---
type: specification
status: implemented_waiting_for_approval
version: 0.1
created: 2026-07-17
updated: 2026-07-17
---

# Agent OS Worker Protocol v0.1

## 目的

AOS-002 用统一协议让 Agent、Skill 和本机工具接收受控工作包，并把结构化状态与可复算证据交回 Agent OS。公开入口只有：

`runner/agent_os_worker.ps1`

协议不负责选择模型、自动创建 Agent、发布内容、修改核心规则或绕过 AOS 运行任务的人工门。

## 三个动作

| 动作 | 输入 | 结果 |
|---|---|---|
| `Prepare` | 运行中的 AOS 任务、适配器、请求 | 创建最小 Worker 快照和 `prepared` 作业 |
| `Accept` | `prepared` 作业、Worker 结果 | 复算权限与证据哈希，接受一个终态结果 |
| `Inspect` | 作业目录 | 从 Worker Ledger 恢复状态并复验输入、回执和证据 |

`Prepare` 只接受 `state=running` 且 Ledger 完整的 AOS 运行任务。规划批准或路由结果不能直接代替运行授权。

## 能力与权限适配器

适配器必须位于 Runner 配置的受信任 `AdapterRoot`（默认 `adapters/`），通过 `agent_os_worker_adapter.schema.json`，并声明：

- `adapter_type`：`agent`、`skill` 或 `local_tool`；
- 每项 `capability` 的上下文槽位、允许根目录、文件数、单文件字节上限、`text_utf8/binary` 内容模式、扩展名和输出类型；
- `context_read=snapshotted_only`；
- 输出、网络和敏感副作用的权限上限。

参考适配器是 `adapters/local-text-worker.json`。适配器只是能力和权限合同，不代表工具已被实际调用。

## 最小上下文规则

Worker 请求必须通过 `agent_os_worker_request.schema.json`。Harness 在创建任何作业目录之前完成以下检查：

1. 请求只绑定能力声明中的槽位；
2. 必需槽位存在，数量不超过 `max_items`；
3. 文件位于适配器允许根目录内，扩展名和实际字节数符合适配器；
4. 请求输出属于能力声明；
5. 所有声明为 `text_utf8` 的上下文均按实际字节解码并检查凭据样式内容，不依赖扩展名推断；
6. 请求权限不超过适配器；v0.1 不接受发布、删除、付款、账号或核心规则修改等敏感副作用；
7. AOS 运行合同、适配器和 Worker 请求由同一份已读字节计算字段与 SHA-256，避免检查后替换；
8. AOS 任务必须位于 Runner 配置的受信任 `RuntimeRoot`，不能从传入任务路径反推信任根。

通过后只复制请求中明确绑定的文件。Worker 输入不包含源文件路径或运行合同路径，只包含作业内快照路径、来源 SHA-256、任务和合同哈希。

## Worker 输入

`worker_input.json` 必须通过 `agent_os_worker_input.schema.json`，核心字段为：

- AOS `task_id`、`task_key`、合同 SHA-256；
- `Prepare` 最终提交时复验的 AOS Ledger 头与序号；
- 适配器 ID、类型、清单 SHA-256；
- Worker 请求 SHA-256；
- 单一能力与目标；
- 上下文快照、来源 SHA-256 和字节数；
- 上下文的显式内容模式；
- 请求输出；
- 本次实际授予的权限。

Worker 只能读取 `context/` 快照，只能在 `output/` 写结果。由于协议要求结构化结果和至少一份证据，每个已准备作业必须授予 `job_output_only` 回传通道；`output_write=none` 不能创建 Worker 作业。协议字段不能自动扩大底层进程沙箱；具体执行器仍必须应用同等或更严的系统权限。

## Worker 回传

Worker 结果必须位于作业 `output/`，并通过 `agent_os_worker_result.schema.json`：

- `status` 只能是 `succeeded`、`failed` 或 `blocked`；
- 必须绑定 `job_id`、适配器、能力和输入 SHA-256；
- `permissions_used` 不得超过准备阶段授予的权限；敏感副作用必须由本协议之外的独立人工批准机制处理；
- 至少提交一份位于 `output/` 的证据；
- 每份证据包含相对路径、类型、`text_utf8/binary` 内容模式和 SHA-256；内容模式由协议根据证据类型和扩展名复核，`report`、`log` 与已知文本扩展名不能伪装为二进制；所有证据字节都执行凭据特征扫描，文本模式还必须通过严格 UTF-8 解码；
- 失败或阻塞原因放入 `errors`。

`failed` 和 `blocked` 至少要提供一条 `errors`；不能用空原因终止。

`Accept` 会对结果与证据持有只读排他写锁，从同一份稳定字节解析字段并计算哈希。Worker 自报哈希不作为事实源。

## 状态与恢复

每个作业目录包含：

```text
worker-job-*/
├── context/
├── output/
├── worker_input.json
├── job.json
├── ledger.jsonl
└── result_receipt.json
```

Worker Ledger 是两段不可变状态链：

```text
prepared → succeeded | failed | blocked
```

每个事件绑定前一事件哈希、输入哈希和请求哈希；终态事件额外绑定结果、回执，以及每份证据的路径、类型、内容模式与哈希。新事件使用 Worker Ledger `schema_version=0.2`。恢复器对 `0.1` 只尝试本项目实际生成过的旧哈希候选：无请求哈希、含请求哈希，以及过渡期证据内容模式纳入或未纳入哈希；这些形态都有固定夹具。旧链的输入、结果和回执只可通过对应的 `legacy_0.1` 严格兼容 Schema，缺失的证据内容模式由受信任协议分类补推；`0.2` 不接受兼容 Schema。版本只允许全 `0.1` 历史链或持有请求哈希绑定的 `0.1 → 0.2` 升级，拒绝 `0.2 → 0.1` 降级；没有请求哈希的旧 `prepared` 作业可只读检查，但在写回执前无副作用地拒绝升级。终态后拒绝再次接收。

`Accept` 使用 `job.lock` 排他接收，回执必须通过 `agent_os_worker_receipt.schema.json` 并用 `CreateNew` 创建，不覆盖已有文件。若崩溃发生在“回执已写、Ledger 未追加”，再次接收只复用 Schema 完整、确定性字段逐项匹配且非链接的回执；`accepted_at` 是首次回执创建时写入并保留的合法时间戳。若发生在“Ledger 已追加、job 快照未更新”，`Inspect` 以 Ledger 为事实源原子重建可变 `job.json`。适配器和能力身份始终取自已哈希的 `worker_input.json`，不信任可变 job 缓存。

`Inspect` 会重新计算输入、回执、结果和证据；不可恢复的漂移显式返回 `broken`，不会覆盖证据。

## 使用示例

```powershell
$prepared = pwsh -File .\10-项目\Hermes-Harness\runner\agent_os_worker.ps1 `
  -Action Prepare `
  -RuntimeTaskPath "C:\path\agent-os-task-AOS-002-*" `
  -AdapterManifestPath ".\10-项目\Hermes-Harness\adapters\local-text-worker.json" `
  -RequestPath "C:\path\worker-request.json" `
  -AsJson
```

Worker 根据 `$prepared.input_path` 执行后，把符合结果 Schema 的 JSON 和证据写入作业 `output/`：

```powershell
pwsh -File .\10-项目\Hermes-Harness\runner\agent_os_worker.ps1 `
  -Action Accept `
  -JobPath "C:\path\worker-job-*" `
  -ResultPath "C:\path\worker-job-*\output\worker_result.json" `
  -AsJson

pwsh -File .\10-项目\Hermes-Harness\runner\agent_os_worker.ps1 `
  -Action Inspect `
  -JobPath "C:\path\worker-job-*" `
  -AsJson
```

## AOS-002 验收映射

- “Worker 只接收最小必要上下文”：声明槽位、数量、字节、扩展名、显式快照和无源路径泄漏共同约束；
- “结果包含结构化状态和证据哈希”：结果 Schema、Harness 复算、终态回执和 Worker Ledger 共同约束；
- “能力声明与权限适配器”：适配器 Schema、参考适配器和准备/回传双向权限检查共同实现。

AOS-002 实现验证通过后仍停在 `waiting_for_approval`。只有用户明确验收，AOS 运行内核才能把它转为 `completed`；不得自动开始 AOS-003。
