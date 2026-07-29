---
title: "Strands 工具系统设计"
tags: [agent-harness, tool-system, strands-agents, json-schema, mcp]
created: 2026-07-29
updated: 2026-07-29
---

# Strands 工具系统设计

> 每个工具是 `AgentTool` ABC 的子类，通过 `stream()` async generator yield 类型化事件，最后一个事件是 `ToolResultEvent`。`@tool` 装饰器从 Python 类型提示 + docstring 自动生成 JSON Schema。

Strands 的工具系统围绕一个核心抽象 `AgentTool` 构建，所有工具（函数装饰器、模块化工具、MCP 工具、结构化输出、子 Agent）都统一为同一接口。工具的执行是**流式事件驱动**的：`stream()` 是一个 async generator，中间 yield 进度事件，最后 yield 一个 `ToolResultEvent` 收尾。这种设计让工具能够天然支持流式输出、中断传播和 trace 追踪。

## 核心抽象：AgentTool

```python
class AgentTool(ABC):
    @property
    @abstractmethod
    def tool_name(self) -> str: ...
    @property
    @abstractmethod
    def tool_spec(self) -> ToolSpec: ...  # {description, inputSchema:{json}, name, outputSchema?}
    @property
    @abstractmethod
    def tool_type(self) -> str: ...  # 'function' | 'python' | 'agent' | 'structured_output'
    @abstractmethod
    async def stream(self, tool_use, invocation_state, **kwargs) -> AsyncGenerator[TypedEvent]: ...
    @property
    def supports_hot_reload(self) -> bool: return False
```

契约：`stream()` 的**最后一个 yield 事件是 `ToolResultEvent`**（包装 `ToolResult = {content, status, toolUseId}`）。中间事件是 `ToolStreamEvent`。

`AgentTool` 定义了三个必填属性（`tool_name` / `tool_spec` / `tool_type`）和一个必填方法（`stream`）。`tool_spec` 中的 `inputSchema` 字段名带 `json` 嵌套层，是为了与 MCP `Tool` 类型对齐——MCP spec 规定 `inputSchema` 是一个 JSON Schema 对象。`supports_hot_reload` 默认为 `False`，只有声明支持热重载的工具才允许在 registry 中被同名覆盖。

## 工具 Spec 生成管线

Strands 有五条平行的管线，把不同来源的"工具定义"统一转换为 `ToolSpec`。

### Pipeline A：@tool 装饰器（自动生成 schema）

```
Python 函数
  → inspect.signature + get_type_hints(include_extras=True)
  → docstring_parser.parse(:param:/Args: 段)
  → _create_input_model():
      - 跳过特殊参数 (self, cls, agent, context_param)
      - 解析 Annotated[T, ...]（pydantic.Field 在 Annotated 中被阻止）
      → pydantic.create_model(field_definitions)
  → input_model.model_json_schema()
  → _clean_pydantic_schema(): 移除 title/additionalProperties，简化 Optional
  → ToolSpec{name, description, inputSchema:{json: schema}}
```

这是开发者体验最好的路径：写一个普通 Python 函数，加 `@tool` 装饰器，类型提示和 docstring 自动变成给 LLM 看的 JSON Schema。管线的关键步骤：

1. **`get_type_hints(include_extras=True)`** 保留 `Annotated` 元数据，用于提取 `Field(...)` 约束（默认值、描述、min/max）。
2. **跳过特殊参数**——`self`/`cls`/`agent`/`context_param` 不会出现在 schema 中，因为这些是框架注入的，不是 LLM 应填的。
3. **Pydantic `create_model`** 动态构建输入模型，`model_json_schema()` 生成标准 JSON Schema。
4. **`_clean_pydantic_schema`** 做减法：Pydantic 默认会给每个字段加 `title`、给对象加 `additionalProperties: false`，这些对 LLM 调用是噪音，移除后 schema 更紧凑。

```python
from strands import Agent, tool

@tool
def word_count(text: str) -> int:
    """Count words in text.

    Args:
        text: The text to count words in.
    """
    return len(text.split())
```

> [!note] Annotated 中的 Field 限制
> 管线中注明 "pydantic.Field 在 Annotated 中被阻止"——这是指 Strands 自己的解析逻辑选择不直接采用 `Annotated[T, Field(...)]` 形式，而是走自己的字段定义构建路径。开发者在写 `@tool` 函数时应使用普通类型提示 + docstring 描述，而非 `Annotated` + `Field`。

### Pipeline B：模块化工具（legacy）

`.py` 文件声明 `TOOL_SPEC` dict + 同名函数 → `PythonAgentTool`。不做 Pydantic 验证，函数直接接收 `tool_use` 原始输入。

这是早期 Strands 的工具写法，现在仍被支持但不推荐用于新代码。`TOOL_SPEC` 是手写的 dict，开发者要自己保证 schema 与函数签名一致——没有自动验证，容易漂移。`PythonAgentTool` 把 `tool_use["input"]` 原样作为 kwargs 传给函数，类型错误不会被拦截。

### Pipeline C：MCP 工具

`MCPAgentTool` 适配 `mcp.types.Tool` → ToolSpec（inputSchema 直接来自 MCP）。`stream()` 调用 `mcp_client.call_tool_async()`。

MCP（Model Context Protocol）工具的 schema 由 MCP server 提供，Strands 直接透传 `inputSchema`，不做二次生成。这使得 Strands 能无缝接入任何符合 MCP 标准的工具服务器。调用时通过 `MCPClient` 的 `call_tool_async` 异步执行，支持 task-augmented 执行（MCP 2025-11-25 spec）。

### Pipeline D：结构化输出

Pydantic BaseModel → `convert_pydantic_to_tool_spec()`：展平 `$ref`、展开嵌套模型、处理 Optional → nullable。`StructuredOutputTool` 验证输入并存储结果。

当 Agent 需要输出结构化数据时，定义一个 Pydantic 模型，Strands 把它转成工具调用——LLM "调用"这个工具来产出结构化结果。`convert_pydantic_to_tool_spec` 处理了 Pydantic schema 中常见的 `$ref`（嵌套模型引用）和 `Optional[T]`（需要变成 `nullable: true`），使 schema 对 LLM 友好。

### Pipeline E：Agent 作为工具

`Agent.as_tool()` → `_AgentAsTool`，单个 `input` 字符串参数。流式传播子 agent 事件，中断向上传递。

这是多 Agent 编排的基石：一个 Agent 可以被注册为另一个 Agent 的工具。子 Agent 的事件（思考、工具调用、工具结果）会向上流式传播到父 Agent 的事件流中。如果父 Agent 收到中断信号，中断会传递给正在运行的子 Agent，实现级联取消。

## ToolRegistry

```python
class ToolRegistry:
    self.registry: dict[str, AgentTool]       # 静态工具
    self.dynamic_tools: dict[str, AgentTool]   # 运行时加载
    self._tool_providers: list[ToolProvider]   # 生命周期管理的工具集合
```

`ToolRegistry` 是工具的集中管理器，区分三类工具来源：

- **`registry`**：静态注册的工具，在 Agent 初始化时确定。
- **`dynamic_tools`**：运行时动态加载（如热重载、MCP 延迟加载），查找时优先于 `registry`。
- **`_tool_providers`**：`ToolProvider` 实例列表，管理需要生命周期的工具集合（主要是 MCP）。

注册规则：

- **精确名称重复报错**——除非该工具 `supports_hot_reload`，否则同名注册直接抛异常。
- **dash/underscore 归一化**——`my-tool` 和 `my_tool` 在归一化后算同名冲突，防止 LLM 因命名风格不同而混淆。
- **输入形式灵活**——接受文件路径字符串、模块路径字符串、dict、已导入模块、`AgentTool` 实例、嵌套可迭代对象、`ToolProvider`、`AgentBase`（自动调用 `as_tool()`）。Registry 内部统一归一化为 `AgentTool`。

Spec 生成时 deep-copy 每个 spec，再经 `normalize_tool_spec` + `validate_tool_spec`：填充 `type`/`properties`/`required`/默认值；**深度上限 64** 防御恶意 MCP schema 导致的递归爆炸。

## 热重载

`_load_tool_module` 加载工具 `.py` 时使用 `sys.modules` 纪律：key 为 `_strands_tool_{name}`（防止工具名 `json` 覆盖 stdlib），父目录仅在 `exec_module` 期间在 `sys.path` 上。

> [!warning] sys.modules 污染风险
> 如果工具模块直接以自身名字（如 `json`）注册到 `sys.modules`，会覆盖标准库导致灾难性后果。Strands 用 `_strands_tool_{name}` 前缀隔离，并在 `exec_module` 结束后从 `sys.path` 移除父目录，避免工具目录污染全局导入路径。

`ToolWatcher` 使用**单例共享 `watchdog.Observer`**（类变量）跨多个 registry 监听同一目录。`MasterChangeHandler` 分发 `on_modified` 到每个 registry 的 `ToolChangeHandler`。

这样设计的好处：多个 Agent 共享同一工具目录时，只启动一个文件系统观察者，避免重复监听和资源浪费。`MasterChangeHandler` 是一个分发器，把文件变更事件路由到所有感兴趣的 registry。

## 工具调用流程

```
event_loop._handle_tool_execution
  → validate_and_prepare_tools()           # 名称正则 ^[a-zA-Z0-9_\-]{1,64}$
  → agent.tool_executor._execute(...)      # ConcurrentToolExecutor (默认) 或 SequentialToolExecutor
      → ToolExecutor._stream_with_trace()  # spans + metrics
          → ToolExecutor._stream():         # 每个工具
              1. lookup: dynamic_tools ?? registry
              2. [retry loop]
                 a. BeforeToolCallEvent hook → interrupts? cancel_tool?
                 b. ExecuteToolStage middleware chain
                    terminal → ctx.tool.stream(tool_use, invocation_state)
                 c. AfterToolCallEvent hook → retry=True?
                 d. yield ToolResultEvent + append tool_results
```

调用流程的关键层次：

1. **`validate_and_prepare_tools`** 在 event loop 层验证工具名合法性。
2. **`tool_executor`** 是可插拔的——默认 `ConcurrentToolExecutor` 并发执行多个 tool_use，`SequentialToolExecutor` 顺序执行。
3. **`_stream_with_trace`** 包裹真正执行，注入 OpenTelemetry spans 和 metrics。
4. **`_stream`** 内部有 retry loop：BeforeToolCallEvent hook 可触发中断或取消；ExecuteToolStage middleware 链最终调用 `tool.stream()`；AfterToolCallEvent hook 可请求 retry。

> [!tip] 并发执行器的顺序保证
> `ConcurrentToolExecutor` 将所有 tool_uses 作为 `asyncio.create_task`，但通过 `asyncio.Queue` + 每任务 `asyncio.Event` 序列化事件发射，保证流式事件仍然按序到达。

这是并发工具执行的核心技巧：工具**执行**是并发的（多个工具同时跑），但**事件发射**是序列化的（通过 Queue + Event 协调）。这样 LLM 收到的事件流保持确定性顺序，同时享受并发带来的延迟降低。

## 参数验证：两层

1. **工具名验证**：正则 `^[a-zA-Z0-9_\-]{1,}$`，最大长度 64。无效名称生成 error ToolResult 反馈给 LLM
2. **输入验证**（仅 @tool 装饰器）：`input_model(**input_data).model_dump()` 通过生成的 Pydantic 模型——强制类型转换，拒绝错误输入

两层验证的分工：工具名验证在 event loop 层（所有工具类型都经过），输入验证在工具内部（只有 `@tool` 装饰器生成的 `FunctionTool` 有 Pydantic 模型）。模块化工具（Pipeline B）和 MCP 工具（Pipeline C）不做输入验证——前者直接传原始 dict，后者由 MCP server 自行验证。

无效工具名不会抛异常中断流程，而是生成一个 `status="error"` 的 `ToolResult` 反馈给 LLM，让 LLM 自行纠正。这是"软失败"设计：保持 agent 循环不中断。

## MCP 集成

`MCPClient(ToolProvider)` 在**守护后台线程**中运行 MCP `ClientSession`：

- `start()` 阻塞在 `_init_future`（初始化完成或超时）
- `load_tools()` 分页 `list_tools_sync()`，应用 prefix 和 tool_filters
- `call_tool_async()` 支持 task-augmented 执行（MCP 2025-11-25 spec）
- `load_servers(config)` 解析 `mcpServers` JSON，自动检测传输（command→stdio, url→streamable-http），插值 `${VAR}` 环境变量
- 生命周期管理：`add_consumer`/`remove_consumer`，最后一个消费者移除时 `stop()`

MCP 集成是 Strands 工具系统中最复杂的部分（`mcp_client.py` 1778 行）。核心设计决策是在**后台线程**运行 MCP session——因为 MCP SDK 的 `ClientSession` 是基于 asyncio 的，而 Strands 的主事件循环也在 asyncio 中，两者不能混用同一 loop。后台线程隔离解决了这个问题。

`load_servers` 的传输自动检测逻辑：config 中有 `command` 字段 → stdio 传输（启动子进程）；有 `url` 字段 → streamable-http 传输（HTTP 连接）。`${VAR}` 插值允许在 config 中引用环境变量，避免硬编码密钥。

## ToolProvider 抽象

```python
class ToolProvider(ABC):
    async def load_tools(self, **kwargs) -> Sequence[AgentTool]: ...
    def add_consumer(self, consumer_id) -> None: ...    # 幂等
    def remove_consumer(self, consumer_id) -> None: ...  # 幂等，零消费者时可能清理
```

生命周期管理的工具集合，资源（后台线程、socket）绑定到使用它们的 agent。由 `MCPClient` 实现。

`ToolProvider` 解决的问题：MCP 工具需要后台线程、socket 等资源，这些资源的生命周期不应该绑定到单个工具实例，而应该绑定到"使用这组工具的 agent 集合"。`add_consumer`/`remove_consumer` 是幂等的引用计数——多个 agent 共享同一个 MCP server 时，只在最后一个 agent 移除时才真正关闭连接。

## 对 llm-harness-runtime 的启示

1. **@tool → Pydantic → JSON Schema** 管线对开发者体验极佳，值得借鉴——开发者只需写类型提示和 docstring，不需要手写 schema。
2. **sys.modules 纪律**防止工具名冲突是重要细节——`_strands_tool_{name}` 前缀隔离是简单有效的方案。
3. **ConcurrentToolExecutor 的 Queue+Event 模式**解决了并发工具的顺序发射问题——执行并发、事件有序，是流式 agent 的实用模式。
4. **ToolProvider 生命周期管理**对 MCP 等需要后台资源的工具集合很重要——引用计数 + 幂等操作避免资源泄漏。
5. **AgentAsTool + 中断传播**使多 agent 编排自然融入工具系统——不需要单独的编排层，工具系统本身就是编排机制。
6. **Schema 深度上限 64** 是防御恶意 MCP schema 的实用措施——外部 schema 不可信，必须有防御性限制。

> [!note] 设计哲学
> Strands 工具系统的核心思想是**统一抽象**：无论工具来源是函数、模块、MCP server、Pydantic 模型还是另一个 Agent，最终都是 `AgentTool` 的 `stream()` 方法。这种统一让工具调用流程、验证、trace、中断传播等横切逻辑只需实现一次。

## 参考

- 源码: `strands-py/src/strands/types/tools.py` (`AgentTool` ABC)
- 源码: `strands-py/src/strands/tools/decorator.py` (33KB, `@tool` 装饰器)
- 源码: `strands-py/src/strands/tools/registry.py` (29KB, `ToolRegistry`)
- 源码: `strands-py/src/strands/tools/mcp/mcp_client.py` (1778行, MCP 集成)
- [[strands-agents-sdk]] — 源摘要
- tool-use-system — 通用 Tool Use 系统设计
- [[strands-hooks-middleware]] — Middleware `ExecuteToolStage`
