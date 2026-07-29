---
title: "Strands 多 Agent / Session / Memory"
tags: [agent-harness, multi-agent, session, memory, strands-agents]
created: 2026-07-29
updated: 2026-07-29
---

# Strands 多 Agent / Session / Memory

> 三种多 Agent 编排模式（Graph/Swarm/A2A）建在共同基类上；Session 是 hook 驱动的对话状态持久化；Memory 是 plugin 驱动的跨会话回忆；Storage 是最小 4 操作字节级 Protocol。

Strands Agents SDK 在单个 `Agent` 之上构建了四个相互正交的子系统：**多 Agent 编排**解决"多个 agent 如何协作"，**Session** 解决"对话状态如何持久化与恢复"，**Memory** 解决"跨对话的知识如何回忆"，**Storage** 则是它们共同依赖的最底层字节存储抽象。理解这四层的关系是理解整个 SDK 运行时的关键。

## 多 Agent 编排

三种模式都继承 `MultiAgentBase`（抽象方法：`invoke_async`、`serialize_state`/`deserialize_state`、`add_hook`）。这一共同基类保证了任何多 agent 模式都可以作为另一个多 agent 模式的节点出现——嵌套是设计上的头等公民，而非事后补丁。

### Graph：确定性 DAG

Graph 是最显式的编排模式，开发者完全控制执行拓扑。

- 节点 = `AgentBase | MultiAgentBase`（支持嵌套：Graph-in-Graph）
- 边 = `GraphEdge(from, to, condition?)`，condition 是 `EdgeCondition | EdgeConditionWithContext`
- 执行：入口点 → 并行批次 → `_find_newly_ready_nodes()` 检查边条件 → 下一批
- 路由语义：fan-in 用 OR（≥1 入边满足即就绪）
- 支持循环（feedback loops）、`reset_on_revisit`、`max_node_executions`/`execution_timeout`/`node_timeout` 限制
- `GraphBuilder` 流式 API：`add_node()` / `add_edge(from, to, condition)` / `set_entry_point()` / `build()`

```python
graph = (
    GraphBuilder()
    .add_node("researcher", research_agent)
    .add_node("writer", writer_agent)
    .add_node("reviewer", reviewer_agent)
    .add_edge("researcher", "writer")
    .add_edge("writer", "reviewer")
    .add_edge("reviewer", "writer",
              condition=EdgeConditionWithContext(lambda ctx: ctx.get("needs_revision")))
    .set_entry_point("researcher")
    .build()
)
```

关键设计点在于 **并行批次**：每一"波"中所有就绪节点并发执行，屏障之后才推进下一波，这天然支持 fan-out/fan-in 而无需手写并发控制。循环通过 `reset_on_revisit` 控制节点状态是否在重入时清空，避免无限循环由 `max_node_executions` 兜底。

### Swarm：自组织协作

Swarm 放弃显式拓扑，转而让 LLM 自行决定控制流。

- 节点 = `Agent` only
- 无显式边；通过注入 `handoff_to_agent` 工具协调
- `SharedContext`：每节点 JSON 可序列化 key-value 存储
- Handoff 流程：agent 调用工具 → `_handle_handoff()` 设置 handoff_node/message → 主循环检测 → 下一个 agent 获取 handoff 消息 + 任务 + 节点历史 + 共享上下文 + 可用 agent 列表
- 完成条件：agent 结束且未调用 handoff
- 限制：`max_handoffs=20`、`max_iterations=20`、`repetitive_handoff_detection`

Swarm 的优雅之处在于：协调逻辑被编码为**工具**而非框架代码。LLM 在常规 tool-use 循环中自然地"决定"何时切换 agent，与调用 `web_search` 或 `calculator` 没有本质区别。`SharedContext` 提供带外通道，避免把状态塞进对话历史。

`repetitive_handoff_detection` 是必要的兜底——LLM 可能在两个 agent 之间无限乒乓球，检测连续重复 handoff 模式后强制终止。

### A2A：Agent-to-Agent 协议

A2A 模式遵循 [a2a-protocol.org](https://a2a-protocol.org) 规范，让 Strands agent 跨进程/跨网络协作：

- **Server**（`A2AServer`）：将 Strands Agent 包装为 HTTP 服务器（Starlette/FastAPI/uvicorn），发布 AgentCard
- **Client**（`A2AAgent`）：继承 AgentBase，可作为 Graph/Swarm 节点
- Context 隔离：`agent_factory(context_id)→Agent`（推荐，每 context 独立 agent+lock，并发，LRU 1000）或 deprecated 单 agent（快照交换，串行）
- 状态映射：A2A TaskState → Strands StopReason（completed/failed/canceled→end_turn；input_required→interrupt）

> [!note]
> A2A 的 `agent_factory` 模式值得借鉴：为每个 context id 分配独立 agent 实例 + 独立锁，既实现并发隔离，又通过 LRU 1000 限制内存膨胀。这比"单 agent + 快照交换"的串行模型更贴合真实多租户场景。

### 三模式对比

|维度|Graph|Swarm|A2A|
|---|---|---|---|
|拓扑|显式 DAG|隐式（工具 handoff）|网络协议|
|控制流|确定性（边条件）|自主（LLM 决定 handoff）|请求-响应|
|节点类型|AgentBase \| MultiAgentBase|Agent only|远程 A2A agent|
|上下文共享|依赖输出传播|SharedContext + handoff 消息|消息传递|
|并行|并行批次|顺序（一次一个 agent）|每 context 并发|
|嵌套|Graph/Swarm 作为节点|否|A2AAgent 作为节点|

选择启发式：**工作流可枚举** → Graph；**协调由 LLM 判断** → Swarm；**跨进程/跨团队** → A2A。三者并非互斥——Graph 节点可以是 Swarm，Swarm 节点可以是 A2AAgent。

### Agent 作为工具

`Agent.as_tool(name?, description?, preserve_context=False)` → `_AgentAsTool`：

- 单个 `input` 字符串参数
- `preserve_context=False`：构造时快照 messages+state，每次调用前重置（无状态）
- `preserve_context=True`：对话历史跨调用保留
- 线程安全：`threading.Lock`（非阻塞；占用时返回 'busy' 错误）
- 中断传播：子 agent `stop_reason='interrupt'` → `ToolInterruptEvent` → 父 agent 处理

这是第四种隐式编排模式：父 agent 通过工具调用间接驱动子 agent。与 Swarm 的区别在于——`as_tool` 是**父 agent 主动调用**且**父 agent 始终保持控制**，而 Swarm 的 handoff 是**平级切换**。

## Session 管理

Session 解决"同一个 agent 的多次调用如何共享对话状态"。

### 接口层次

```
HookProvider
  └─ SessionManager (ABC)          ← 生命周期 hook 接线
       └─ RepositorySessionManager  ← 变更检测，消息管理
            ├─ FileSessionManager   ← 本地文件系统
            └─ S3SessionManager     ← S3 对象

SessionRepository (ABC)            ← Session/SessionAgent/SessionMessage 的 CRUD
```

### Hook 接线

SessionManager 是 HookProvider，注册在 agent 生命周期事件上：

- `AgentInitializedEvent` → `initialize()`：创建或恢复 agent
- `MessageAddedEvent` → `append_message()` + `sync_agent()`
- `AfterInvocationEvent` → `sync_agent()`
- 多 agent 事件 → `sync_multi_agent()`

> [!tip]
> Session 持久化对 agent 业务代码**完全透明**——agent 只管正常发消息、调用工具，SessionManager 通过 hook 在生命周期事件上自动落盘。这是 HookProvider 模式的最佳实践示范。

### 数据模型

- `Session`：session_id, session_type, timestamps
- `SessionAgent`：agent_id, state（用户态）, conversation_manager_state, _internal_state（interrupt_state, model_state）
- `SessionMessage`：message, message_id（索引）, redact_message, timestamps
- `encode_bytes_values`/`decode_bytes_values`：base64 往返二进制内容

`_internal_state` 的存在说明 Session 不仅持久化用户可见消息，还持久化**框架内部状态**（如 interrupt 的挂起状态、model 的缓存状态），这样才能在恢复后从中断点继续。

### 优化

- **版本变更检测**：`sync_agent()` 比较状态版本号，未变则跳过 I/O
- **`_fix_broken_tool_use()`**：修复恢复历史中的孤儿/陈旧 toolUse-toolResult 对
- **原子写入**：FileSessionManager 用 mkstemp+os.replace；symlink 保护；目录权限 0o700

`_fix_broken_tool_use()` 是个值得注意的防御性设计：序列化/恢复过程中可能产生孤儿 toolUse（没有对应 toolResult）或陈旧配对，这会让模型上下文不一致。版本变更检测则把"无变化的 sync"从 O(消息数) 降为 O(1) 比较操作。

## Memory 系统

> [!key-insight] Session vs Memory 的关键区别
> - **Session**：持久化*当前对话状态*（消息、agent 状态），同一 agent 的调用间连续。Hook 驱动，同步，即时持久化。
> - **Memory**：提供*跨对话回忆*（事实、偏好），来自*之前的对话*。Plugin 驱动，异步，后台提取。语义搜索蒸馏知识。

这一区分是 Strands 设计中的核心洞见。许多框架把"记忆"和"对话历史"混为一谈，导致要么把全部历史塞进上下文（爆 token），要么丢失跨会话知识。Strands 用两套独立子系统分别处理，各自优化。

### MemoryManager（继承 Plugin）

管理 `list[MemoryStore]`，三种行为：

1. **工具**：`search_memory`（始终注册）和 `add_memory`（可选）。fire-and-forget 模式可选
2. **提取**（后台）：`ExtractionCoordinator` 缓冲所有消息。触发时：
   - 有 extractor（如 ModelExtractor）：模型调用蒸馏事实 → 写入各 store
   - 无 extractor：原始过滤消息 → `store.add_messages()`（服务端提取）
   - 每 store 高水位线（至少一次交付），每 store 任务链串行化，10 次失败后退避
   - 触发器：`InvocationTrigger`（每轮）、`IntervalTrigger(N)`（每 N 轮，默认 5）
3. **注入**：`InvokeModelStage.Input` middleware 将检索到的记忆折叠到模型输入。自适应查询：user turn 用最新用户消息文本，否则用最新 assistant 文本。默认 `<memory>` XML 块格式

> [!note]
> MemoryManager 同时是 Plugin、HookProvider（提取）、Middleware（注入）——一个对象横跨三个关注点。这种"行为聚合"模式比拆成三个独立组件更内聚，因为三者共享同一组 MemoryStore 配置。

### MemoryStore Protocol

```python
class MemoryStore(Protocol):
    name: str
    description: str | None
    max_search_results: int | None
    writable: bool
    extraction: ExtractionConfig | bool | None

    async def search(query, options?) -> list[MemoryEntry]  # 必须
    async def add(content, metadata?) -> Any                 # 可选（可写 store）
    async def add_messages(messages, context?) -> Any        # 可选（服务端提取）
    async def initialize() -> None                           # 可选
    def get_tools() -> list[AgentTool]                       # 可选
```

Protocol 设计让 store 实现可以极简——只有 `search` 是必须的。只读 store（如 Bedrock KnowledgeBase）可以不实现 `add`；服务端提取型 store 可以不实现 `get_tools`。

### 内置 Store

- **BedrockKnowledgeBaseStore**：语义搜索 via Bedrock Retrieve，支持 CUSTOM/S3 数据源接入，scope 过滤，ACL 支持。默认只读
- **TestMemoryStore**：零基础设施 JSON 文件 store，词法搜索（token overlap），去重。跨 SDK 兼容格式

## Storage 抽象

```python
@runtime_checkable
class Storage(Protocol[ListQuery]):
    async def write(key: str, data: bytes) -> None
    async def read(key: str) -> bytes | None
    async def delete(key: str) -> None
    async def list(query: ListQuery) -> list[str]
```

- Key 是不透明字符串；内置 backend 把 '/' 当分隔符（折叠，拒绝 '..'）
- `_NamespacedStorage`：可组合前缀视图
- Backend：InMemoryStorage（dict+lock）、LocalFileStorage（原子文件写入）、S3Storage（lazy boto3，分页 list）
- Storage 被 TestMemoryStore 使用，但 Session 管理**不使用** Storage（File/S3SessionManager 直接实现 SessionRepository）

> [!warning]
> 注意 Storage 与 SessionRepository 的关系：Session **绕过** Storage 直接实现 Repository。这是一个有意的不一致——Storage 的字节级语义对 Session 的结构化 CRUD 太低层。不要试图用 Storage 重新实现 Session。

## 层叠关系

```
Agent
├── Tools (tool_registry) ← 含 AgentAsTool, MemoryManager 工具
├── Hooks (HookRegistry)
│    ├── SessionManager (as HookProvider) ← 对话持久化
│    └── MemoryManager extraction (MessageAddedEvent → 缓冲)
├── Middleware (_middleware_registry)
│    └── MemoryManager injection (InvokeModelStage.Input)
├── Plugins (PluginRegistry)
│    └── MemoryManager (Plugin) ← search/add 工具 + 提取 + 注入
└── Session (SessionManager) ← File/S3 backend
```

这张图展示了 Strands 的组装式架构：同一个 `Agent` 实例可以同时挂载 Tools、Hooks、Middleware、Plugins、Session，各子系统通过各自机制接入而互不干扰。`MemoryManager` 是横跨三层的典型（Hooks 提取 + Middleware 注入 + Plugin 工具），证明这种分层足以表达复杂关注点。

## 对 llm-harness-runtime 的启示

1. **Graph 的并行批次 + 边条件**比线性 pipeline 更灵活，适合复杂工作流
2. **Swarm 的 handoff 工具**是自组织多 agent 的优雅模式——LLM 自行决定何时切换
3. **Session 作为 HookProvider** 使持久化透明接入 agent 生命周期
4. **Session vs Memory 分离**很重要：对话连续性 vs 跨对话知识，不同持久化策略
5. **Memory 的后台提取 + 高水位线**保证至少一次交付，是异步记忆系统的关键模式
6. **Storage Protocol 极简**（4 操作）是好的底层抽象——让上层（memory store）决定语义

> [!tip] 借鉴优先级
> 对本仓库最直接可借鉴的是：(1) HookProvider 接线生命周期事件做透明持久化的模式；(2) Memory 的提取-注入分离 + 高水位线至少一次交付；(3) Storage 4 操作 Protocol 的极简设计。Graph/Swarm/A2A 三模式则取决于本仓库是否需要多 agent 能力。

## 参考

- 源码: strands-py/src/strands/multiagent/ (graph.py 65KB, swarm.py 46KB)
- 源码: strands-py/src/strands/session/ (session_manager.py, file/s3 backends)
- 源码: strands-py/src/strands/memory/ (memory_manager.py 34KB)
- 源码: strands-py/src/strands/storage/ (storage.py)
- [[strands-agents-sdk]] — 源摘要
- subagent-dispatch — 通用 Subagent 调度
- context-management — 上下文管理
