---
title: "Strands 沙箱 / 安全 / 可观测"
tags: [agent-harness, sandbox, telemetry, safety, strands-agents]
created: 2026-07-29
updated: 2026-07-29
---

# Strands 沙箱 / 安全 / 可观测

> [!note] 沙箱是 6 方法 ABC（execute/code/file ops），4 个后端（Docker/SSH/Local/PosixShell）。可观测性基于 OpenTelemetry，14 个 metrics + GenAI 语义约定 tracing。安全层还有 Guardrails 类型定义、Cedar 策略、HITL 审批、注入系统、检查点和导向。

## 沙箱抽象

```python
class Sandbox(ABC):
    @abstractmethod
    async def execute_streaming(self, command, *, timeout, cwd, env, **kwargs) -> AsyncGenerator[StreamChunk | ExecutionResult]: ...
    @abstractmethod
    async def execute_code_streaming(self, code, language, *, timeout, cwd, env, **kwargs) -> AsyncGenerator[...]: ...
    @abstractmethod
    async def read_file(self, path, **kwargs) -> bytes: ...
    @abstractmethod
    async def write_file(self, path, content: bytes, **kwargs) -> None: ...
    @abstractmethod
    async def remove_file(self, path, **kwargs) -> None: ...
    @abstractmethod
    async def list_files(self, path, **kwargs) -> list[FileInfo]: ...

    def get_tools(self) -> list[AgentTool]: ...  # 工具自动注册
```

6 个抽象方法覆盖命令执行、代码执行、文件读写/删除/列举三大类能力。`get_tools()` 是非抽象的便利方法：后端只需实现基础原语，工具层（`make_bash` / `make_file_editor`）自动绑定到 `self`，Agent 在初始化时通过 `self._sandbox = sandbox or NotASandboxLocalEnvironment()` 获得沙箱实例。

### 4 个后端

| 后端 | 基类 | 隔离 | 用途 |
|---|---|---|---|
| DockerSandbox | PosixShellSandbox | ✅ 容器 | `docker exec [--user] [-w] [-e]... -- container sh -c cmd` |
| SshSandbox | PosixShellSandbox | ✅ 远程主机 | `ssh -o BatchMode=yes -- host 'cd CWD && cmd'` |
| NotASandboxLocalEnvironment | Sandbox | ❌ 无 | 默认回退，`sh -c` 本地执行 |
| PosixShellSandbox | Sandbox (ABC) | 取决于子类 | shell 命令基础设施 |

`PosixShellSandbox` 是 Docker 与 SSH 的共享基类，封装了 POSIX shell 下通用的进程监督、流式输出聚合、超时与信号处理逻辑。Docker 与 SSH 子类只负责把"如何拼接命令行前缀"这一层差异实现掉。

### 安全细节

- **Docker**：`--` 终止 flag 解析（防止容器名作为 flag 注入）；env 用 `-e` argv（无需 shell 引号）
- **SSH**：`_ALLOWED_SSH_OPTIONS` 白名单（37 个安全选项，排除命令执行/隧道）；`--` 防止 host-as-flag 注入；`StrictHostKeyChecking=accept-new` 默认
- **环境变量验证**：`ENV_KEY_PATTERN=^[a-zA-Z_][a-zA-Z0-9_]*$`（POSIX 合法名）
- **解释器名验证**：`LANGUAGE_PATTERN=^[a-zA-Z0-9._-]+$`，必须用 `re.fullmatch`（不是 `re.match`）防止换行注入
- **代码执行**：base64 编码 + 引号 heredoc + 管道到解释器（`base64 -d << 'EOF' | lang`）
- **进程监督**：`_stream_process` 用 `start_new_session=True`（独立进程组），超时用 `os.killpg(SIGKILL)` 杀整树

> [!warning] `re.fullmatch` 而非 `re.match` 是关键细节
> `re.match` 只在开头匹配，尾部允许任意字符——包括 `\n`。攻击者可在语言名中嵌入换行逃逸到第二行命令。`re.fullmatch` 强制整串匹配，是防止此类注入的正确做法。

### 沙箱 ↔ 工具集成

Agent 初始化时 `self._sandbox = sandbox or NotASandboxLocalEnvironment()`（每 agent 独立）。自动注册 `sandbox.get_tools()`（跳过用户已注册的同名工具）。Docker/SSH 沙箱自动提供 `make_bash` + `make_file_editor` 绑定到自身。

> [!tip] 工具注册的冲突策略
> 当用户已显式注册同名工具时，沙箱不会覆盖。这让用户可以为本地环境定制 `bash` 工具行为，同时保留 Docker/SSH 沙箱的默认绑定。

## 可观测性

### Metrics（14 个 OTel 仪器）

- **Counter**：cycle_count, start_cycle, end_cycle, tool.call_count, tool.success_count, tool.error_count
- **Histogram**：event_loop.latency(ms), cycle_duration(s), input/output.tokens, cache_read/cache_write.input.tokens, tool.duration(s), model.time_to_first_token(ms)

Counter 关注"发生了多少次"，Histogram 关注"耗时多少/用量多少"。token 维度细分为 input/output/cache_read/cache_write，配合 GenAI 语义约定可对成本与缓存命中率做精确归因。

### Tracing（GenAI 语义约定）

Span 层次：agent → cycle → model/tool

- `invoke_agent {name}` — agent 调用
- `chat` — 模型推理（gen_ai.request.model, gen_ai.usage.* tokens）
- `execute_tool {name}` — 工具调用
- `execute_event_loop_cycle` — 循环周期
- Memory: search/add/inject/extract span
- **Redaction**：`gen_ai.input.messages`、`gen_ai.output.messages`、`gen_ai.system_instructions` 默认 `[REDACTED]`，除非通过 `OTEL_SEMCONV_STABILITY_OPT_IN=gen_ai_unredacted_attributes=<list>` 允许

> [!note] Redaction 默认开启
> 模型输入/输出默认脱敏，防止对话内容泄露到 trace 后端。需要明文调试时，按属性白名单 opt-in，而不是全局开关——降低误开风险。

### 配置

`StrandsTelemetry` 链式 API：

```python
StrandsTelemetry()
    .setup_console_exporter()
    .setup_otlp_exporter()
    .setup_meter(enable_console_exporter=True, enable_otlp_exporter=True)
```

读取 `OTEL_SERVICE_NAME`（默认 `strands-agents`），失败记录日志不抛异常。链式 API 让用户按需选择 console（本地调试）/ OTLP（远端 collector）/ meter 的组合，互不干扰。

## 注入系统

将即时文本折叠到模型输入中，**不修改持久化历史**：

`_create_injection_middleware(render_content, trigger)` 构建 `InvokeModelStage.Input` handler：

1. 构建 `InjectionContext`（messages 防御性拷贝）
2. 按 trigger 门控：`'userTurn'`（仅新用户提问）或 `'everyTurn'`（所有模型调用）
3. `_fold_into_last_user_message()`：普通提问**前置**（用户提问留最后），tool-result 回合**后置**（tool result 必须留第一块）
4. **Fail-open**：trigger/render 异常记录日志并跳过，模型调用继续

`_xml.py` 提供最小 XML 转义（`&`, `<`, `>`），防止存储型 prompt injection。注入内容被折叠进最后一条 user message 而非新增 message——既保持对话结构不变，又让模型自然读到注入文本。

> [!tip] 顺序是有意义的
> 普通提问前置 + 用户提问留最后：模型在"看到注入背景 → 回答用户"的顺序下更稳定。Tool-result 回合后置：因为 tool result 必须留作 messages 第一块，注入文本只能接在后面。

## Agentic Context 管理

`context_manager='agentic'` 注入 3 个工具 + 1 个 middleware：

- `summarize_context(keep_recent, summary_ratio, message_type)` — 压缩最旧消息为摘要
- `truncate_context(keep_recent, message_type)` — 直接丢弃最旧消息
- `pin_context(select, filter, action)` — 固定/取消固定消息
- `create_token_usage_middleware()` — 在最后消息追加 `<context-status>` 块（used/remaining tokens vs context_window_limit）

模型通过 `<context-status>` 实时感知上下文余量，再自行决定调用 `summarize_context` / `truncate_context` 压缩——这是一种"让模型管理自身上下文窗口"的自治范式。`pin_context` 保证关键消息（如系统指令、重要决策）不被压缩误删。

## 内置工具

| 工具 | 工厂 | 说明 |
|---|---|---|
| bash | `make_bash(*, sandbox, name, description)` | `sandbox.execute(command, timeout)`，返回 BashOutput |
| file_editor | `make_file_editor(*, sandbox, name, description)` | view/create/str_replace/insert，拒绝非绝对路径和 `..` 遍历，1MB 上限 |
| http_request | `make_http_request(*, client)` | 流式响应，cancel_signal 检查，>=400 抛 HttpRequestError |
| sleep | `make_sleep(*, max_duration=60)` | asyncio.sleep，协作取消 |

`file_editor` 的路径校验（绝对路径 + 禁 `..`）是与 permission-model 一致的安全基线。`http_request` 的流式 + cancel_signal 让长响应可被中途取消，避免阻塞 event loop。

## 内置插件

| 插件 | 说明 |
|---|---|
| GoalLoop | 迭代精炼：AfterInvocationEvent 验证响应，失败则反馈+resume 重入循环。NL goal 由独立 judge Agent 判断 |
| AgentSkills | 技能集成：`skills` 工具按名激活，BeforeInvocationEvent 注入技能元数据 XML 到 system prompt |
| SteeringHandler | 导向：Proceed/Guide/Interrupt 动作。LLM steering agent 仅评估 context-data 模式（ledger），不评估外部知识 |
| ContextInjector | 包装 `_create_injection_middleware` |
| ContextOffloader | AfterToolCallEvent hook：超大 tool result（>2500 token）卸载到 storage，替换为预览 + 引用 |

GoalLoop 的"独立 judge Agent"避免主 Agent 自我评估的偏见。SteeringHandler 仅评估 ledger（结构化决策记录）而非原始 context，既降低 token 成本也减少外部知识污染判断的风险。

## 检查点

`Checkpoint(frozen dataclass)`：`position: 'after_model' | 'after_tools'`，`cycle_index`。在 ReAct 周期边界发出（仅 tool_use 周期）。**不捕获对话状态**（需配对 SessionManager）。优先级：Interrupt > checkpoint > cancel。恢复通过 `{'checkpointResume': {'checkpoint': ckpt.to_dict()}}`。

> [!warning] Checkpoint 不等于 Session 快照
> Checkpoint 只记录"在哪个周期边界停下了"，不序列化 messages 等对话状态。要完整恢复需要 SessionManager 配合。优先级 Interrupt > checkpoint > cancel 意味着用户中断总是优先于自动恢复点。

## 对 llm-harness-runtime 的启示

1. **沙箱 ABC + 多后端**模式让安全隔离可插拔——本地开发用 NotASandbox，生产用 Docker/SSH
2. **`--` 防 flag 注入**是重要的安全细节，容易遗漏
3. **OTel GenAI 语义约定**是行业标准，应直接采用
4. **注入系统的 fail-open** 策略务实——记忆/上下文注入失败不应阻断模型调用
5. **Agentic context 管理**让模型自行决定何时压缩——创新但需评估可靠性
6. **ContextOffloader** 的预览+引用模式解决了大 tool result 占用上下文的问题
7. **Steering 仅评估 ledger 模式**而非外部知识——重要的安全约束

这些启示与 harness-architecture 的可插拔原则一致，可作为沙箱/可观测子系统的设计参考。Guardrails 类型定义、Cedar 策略、HITL 审批等安全层的进一步细节见 permission-model 和 hook-system。

## 参考

- 源码: strands-py/src/strands/sandbox/ (base.py, docker.py, ssh.py, posix_shell.py)
- 源码: strands-py/src/strands/telemetry/ (tracer.py 54KB, metrics.py 25KB)
- 源码: strands-py/src/strands/injection/ (_message_injection.py)
- 源码: strands-py/src/strands/vended_tools/ (bash, file_editor, http_request, sleep)
- 源码: strands-py/src/strands/vended_plugins/ (goal, skills, steering, context_injector, context_offloader)
- [[strands-agents-sdk]] — 源摘要
- permission-model — 权限模型
- hook-system — Hook 系统
