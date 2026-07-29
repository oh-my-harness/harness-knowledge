---
title: "Strands Agent Loop 架构"
tags: [agent-harness, agent-loop, strands-agents, async-generator]
created: 2026-07-29
updated: 2026-07-29
---

# Strands Agent Loop 架构

> Strands SDK 的核心循环用递归 async generator 而非 while-loop 实现 ReAct 循环，将模型推理、工具执行、上下文管理、中断/取消、重试等职责通过 hooks + middleware 解耦。

## 调用链

```
Agent.__call__(prompt)
  → run_async → invoke_async → stream_async
    → _run_loop(messages, state)
      → _execute_event_loop_cycle(state)
        → event_loop_cycle(agent, state)  # 单轮
```

`Agent.__call__` 是对外入口，同步路径走 `run_async`，异步路径走 `invoke_async`，两者最终都汇入 `stream_async`——整个 agent 的产出物本质是一个 **async generator**，调用方按需消费事件流。`_run_loop` 负责整个会话生命周期的编排，`_execute_event_loop_cycle` 驱动单轮推理，`event_loop_cycle` 才是真正的单轮逻辑实现。

## _run_loop：调用生命周期

```python
while current_messages is not None:
    BeforeInvocationEvent          # hooks 可 cancel 或修改 messages
    _append_messages(*current_messages)
    _execute_event_loop_cycle(invocation_state)
    AfterInvocationEvent           # hooks 可设 .resume 实现自动循环
    current_messages = after_event.resume or None
```

`_run_loop` 是外层 while 循环，但循环的推进并不靠显式 state machine，而是靠 `AfterInvocationEvent.resume`：

- **BeforeInvocationEvent**：hooks 可在此 cancel 整次调用，或改写即将追加的 `messages`，实现 prompt 注入、权限拦截等。
- **追加消息**：把 `current_messages` 写入对话历史，进入单轮执行。
- **_execute_event_loop_cycle**：驱动模型推理 + 工具执行，可能递归多轮（见下文）。
- **AfterInvocationEvent**：hooks 通过设置 `event.resume` 返回下一批要追加的消息，从而实现"自动续聊"——比如外部编排器根据结果决定是否继续。若 `resume` 为 `None`，循环结束。

> [!note] resume 语义
> `resume` 不是重试，而是"外部决定继续"。它让 agent 循环可以从外部被驱动，而不必在循环内部硬编码多轮策略。

## event_loop_cycle：单轮逻辑

```python
1. 检查限制 (turns/tokens caps) → 超限则返回
2. yield StartEvent + StartEventLoopEvent
3. 确定 stop_reason:
   a. interrupt_state.activated → 复用存储的 message
   b. 最新 message 含 toolUse → 复用
   c. 否则调用 _handle_model_execution()
4. yield ModelMessageEvent(message)
5. 按 stop_reason 分支:
   - tool_use → _handle_tool_execution() → recurse_event_loop() → 新一轮
   - end_turn → yield EventLoopStopEvent, return
   - max_tokens → raise MaxTokensReachedException
   - interrupt → yield EventLoopStopEvent('interrupt', interrupts=[...])
```

单轮逻辑分三阶段：**前置检查 → 模型推理（或复用）→ stop_reason 分发**。

步骤 3 是关键——并非每轮都调用模型。当中断恢复（`interrupt_state.activated`）或最新消息已包含 `toolUse` 块时，直接复用已有 message 跳过模型调用，避免重复推理。只有真正需要新决策时才进入 `_handle_model_execution()`。

步骤 5 的 `recurse_event_loop()` 是递归入口：工具执行完成后，把 `tool_result` 追加到历史，然后递归调用 `event_loop_cycle` 开始下一轮，让模型看到工具结果并决定下一步。

## 关键设计模式

### 1. 递归而非循环

每个 tool_use 周期通过 `recurse_event_loop()` 递归进入新的 `event_loop_cycle()`。这意味着每轮的上下文自然通过 async generator 的调用栈传递，而非维护一个显式的 state machine 状态变量。

递归带来的好处：

- **上下文天然隔离**：每轮的局部变量（`tool_use_id`、`tool_results`、`retry_count`）都在各自栈帧中，无需命名空间管理。
- **中断/恢复语义清晰**：中断时只需把当前栈帧状态序列化，恢复时重建栈帧即可。
- **事件流自然嵌套**：递归产生的子事件流可以直接 `yield from` 到外层，无需显式队列。

> [!warning] 栈深度
> Python 默认递归上限 1000，长对话（几百轮工具调用）可能逼近上限。实践中需要监控或调高 `sys.setrecursionlimit`。

### 2. Hook 驱动的重试/取消

重试不是内联逻辑——`ModelRetryStrategy` 是一个 `HookProvider`，注册在 `AfterModelCallEvent` 上：
- 指数退避：4s→8s→16s→32s→64s，上限 240s
- 默认只重试 `ModelThrottledException`
- 通过 `event.retry = True` 触发重试

这种设计把"重试策略"从"重试机制"中剥离：循环本体只知道"hook 可以标记 retry"，而退避曲线、重试哪些异常、最大次数都由 `ModelRetryStrategy` 独立配置，甚至可替换。

取消通过 `threading.Event` 在 4 个安全点检查：流式过程中、工具执行前、工具执行中、工具执行后下一轮前。取消是**协作式**的——循环不会硬中断正在执行的代码，而是在预定义的检查点主动退出，保证资源清理。

### 3. 上下文溢出：反应式 + 主动式

- **反应式**：捕获 `ContextWindowOverflowException` → `conversation_manager.reduce_context(e=e)` → 递归重试
- **主动式**：`BeforeModelCallEvent` hook 比较 `projected_input_tokens / context_window_limit` 与阈值（默认 0.7），超限则主动压缩
- **max_tokens 恢复**：将截断消息中的 toolUse 块替换为文本错误块

反应式处理"模型拒绝了大输入"的异常情况；主动式在调用前就压缩，避免浪费一次失败的 API 调用。`max_tokens` 恢复更精细——当输出被截断时，把未完成的 `toolUse` 块（可能是残缺 JSON）替换为文本错误块，避免下游工具解析失败。

### 4. 流式状态机

`process_stream()` 是有状态的状态机，消费 `model.stream()` 的 chunk，组装 Message 同时 yield 类型化事件：
- `ModelStreamChunkEvent` — 原始 chunk 透传
- `TextStreamEvent` / `ToolUseStreamEvent` / `ReasoningTextStreamEvent` — 类型化 delta
- `ModelStopReason` — 最终事件 (stop_reason, message, usage, metrics)

不同 provider 的流式协议差异巨大（有的按 token，有的按块，有的带 reasoning delta）。`process_stream()` 把这些差异吸收成一个统一的状态机：输入是异构 chunk，输出是类型化事件流 + 组装好的 `Message` 对象。

> [!tip] handle_message_stop 会将 end_turn 覆盖为 tool_use
> 某些模型在包含 toolUse 块时错误返回 end_turn。`streaming.py` 的 `handle_message_stop` (L350) 检测到 toolUse 块时会将 stop_reason 覆盖为 tool_use。

### 5. 中断 (Interrupt)

三种入口：`BeforeToolCallEvent.interrupt()`、`ExecuteToolContext.interrupt()`（middleware）、tool 抛出 `InterruptException`。所有中断共用 agent 上的 `_InterruptState`：
- 中断时保存 `{tool_use_message, tool_results}` 到 context
- 恢复时调用方传入 `interruptResponse` → 跳过 model call，复用存储的 tool_use_message，继续工具执行
- 中断可序列化 (`to_dict`/`from_dict`)，支持跨会话恢复

中断是 Strands 的"human-in-the-loop"核心：工具执行可被暂停，等待人类输入后恢复。三种入口覆盖了不同层级——hook 层（执行前拦截）、middleware 层（执行中拦截）、工具内部（主动抛异常）。

### 6. 并发控制

`_ConcurrencyController` 支持 THROW 模式（默认，同时只允许一个调用）和 UNSAFE_REENTRANT 模式。THROW 模式还支持幂等令牌：相同令牌的调用等待结果复用，不同令牌抛 `ConcurrencyException`。

THROW 模式保护 agent 的有状态上下文不被并发调用破坏；幂等令牌让"重复提交同一请求"变成"等待第一次的结果"而非报错，适合前端重试场景。

## Stop Reason 处理

|Stop Reason|处理|
|---|---|
|`tool_use`|执行工具，然后递归进入下一轮|
|`end_turn`|正常完成|
|`max_tokens`|抛 `MaxTokensReachedException`，调用方可重新 invoke 继续|
|`interrupt`|工具触发中断，停止并返回中断列表|
|`cancelled`|`cancel()` 被调用，在下一个安全点停止|
|`checkpoint`|检查点启用，在 after_model 或 after_tools 边界暂停|
|`limit_turns` / `limit_total_tokens` / `limit_output_tokens`|预算上限命中|

`stop_reason` 是整个循环的"路由表"——每个值对应一条明确的控制流路径。`checkpoint` 值得注意：它让循环可以在模型推理后或工具执行后暂停，把状态序列化，稍后从检查点恢复，实现长任务的断点续跑。

## 工具执行错误反馈

工具错误**被转换为 ToolResult 并反馈给模型**——循环不会因工具失败而中断：
```python
except Exception as error:
    yield ToolResultEvent({
        'toolUseId': tool_use.get('toolUseId'),
        'status': 'error',
        'content': [{'text': f'Error: {error}'}]
    })
```
错误结果组装成 `tool_result_message`（role='user'）追加到历史，然后递归让模型看到错误并调整。

这是 agent 鲁棒性的关键设计：工具失败不是"异常"，而是"给模型的信息"。模型看到 `status: 'error'` 后可以换一种工具调用方式、修正参数、或放弃改用其他策略。只有模型层异常（throttle、context overflow）才会触发重试或压缩。

## ConversationManager 策略

|策略|行为|
|---|---|
|SlidingWindowConversationManager|截断最旧 tool result（保留首尾 200 字），然后裁剪到 window_size，find_valid_trim_point 确保不在孤儿 toolResult/toolUse 处切断|
|SummarizingConversationManager|摘要最旧 30% 消息，用 model.stream() 直接调用避免重入 pipeline 死锁|
|NullConversationManager|用于 stateful 模型（服务端管理），溢出时重新抛出异常|
|`'auto'`|组合 Summarizing + ContextOffloader 插件|
|`'agentic'`|注入 summarize/truncate/pin 工具让模型自行管理上下文|

`SlidingWindow` 的 `find_valid_trim_point` 是个细节但重要的设计：如果裁剪点恰好切在一个 `toolUse` 和它的 `toolResult` 之间，会产生"孤儿"消息（有 toolResult 但没有对应的 toolUse），导致模型困惑。该函数会回退到上一个有效边界。

`Summarizing` 直接调用 `model.stream()` 而非走 agent pipeline，是为了避免重入死锁——agent 循环内部触发的上下文压缩如果再走 agent 循环，会递归触发压缩。

## 对 llm-harness-runtime 的启示

1. **递归 async generator** 比 while-loop 更自然地传递上下文，但要注意栈深度（长对话）
2. **Hook 驱动重试** 将策略与机制解耦——我们应考虑类似模式
3. **流式状态机** 是处理多 provider delta 的干净方式
4. **中断状态序列化** 是跨会话恢复的关键能力
5. **主动+反应式上下文管理** 双层策略值得借鉴

## 参考

- 源码: strands-py/src/strands/event_loop/event_loop.py (883行)
- 流式: strands-py/src/strands/event_loop/streaming.py (535行)
- 重试: strands-py/src/strands/event_loop/_retry.py
- [[strands-agents-sdk]] — 源摘要
- [[strands-hooks-middleware]] — Hooks/Middleware/Interventions 三层扩展
- [[harness-architecture]] — 通用 Harness 架构设计
