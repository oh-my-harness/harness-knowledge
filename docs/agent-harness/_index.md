---
title: Agent Harness
tags: [agent-harness, harness, index]
created: 2026-06-10
updated: 2026-07-29
---

# ⚙️ Agent Harness

> Agent 运行时与 Harness 开发相关知识

## 目录索引

- [Harness 架构设计](harness-architecture.md) — 运行时架构核心模式
- [Tool Use 系统](tool-use-system.md) — 工具调用与注册机制
- [Subagent 调度](subagent-dispatch.md) — 子 Agent 调度与编排
- [Worktree 管理](worktree-management.md) — 隔离工作区方案
- [权限模型](permission-model.md) — 安全与授权框架
- [Context 管理](context-management.md) — 上下文窗口策略
- [Hook 系统](hook-system.md) — 扩展点与事件系统

## Strands Agents SDK — 架构学习笔记

> [strands-agents/](strands-agents/) — 开源 AI Agent SDK（Python + TypeScript）的完整架构学习

- [[strands-agents-sdk|Strands Agents SDK（源摘要）]] — 6 大子系统的完整架构学习。
- [[strands-agent-loop|Agent Loop 架构]] — 递归 async generator 实现的 ReAct 循环，hook 驱动重试/取消，反应式+主动式上下文管理。
- [[strands-model-providers|多模型 Provider 抽象]] — ABC + 13 个 provider，Bedrock-modeled StreamEvent 统一格式，两层 token 计数。
- [[strands-tool-system|工具系统设计]] — @tool 装饰器 → Pydantic → JSON Schema；ToolRegistry + 热重载 + MCP + AgentAsTool。
- [[strands-hooks-middleware|Hooks / Middleware / Interventions 三层扩展]] — 事件回调 / 流式管道 / 类型化决策，Cedar 策略 + HITL。
- [[strands-multi-agent-session|多 Agent / Session / Memory]] — Graph/Swarm/A2A 编排 + 会话持久化 + 跨会话记忆 + Storage 抽象。
- [[strands-sandbox-safety|沙箱 / 安全 / 可观测]] — Docker/SSH/Local 沙箱 + OTel 14 metrics + 注入系统 + 检查点 + 导向。

## 项目笔记

> [projects/](projects/) — 开源系统与框架的架构分析

- [[deeptutor|DeepTutor]] — HKU 开源智能体化辅导框架：混合个性化引擎 + 闭环任务辅导，TutorBench 评估 +10.76%，通用推理 +29.4%。
- [[fluxeda|FluxEDA]] — 浙江大学：统一有状态基础设施，使 LLM 智能体在商业 EDA 工具上执行多步迭代优化，支持状态回滚和分支探索。

## HarnessX — 架构学习笔记

> [harnessx/](harnessx/) — 可组合、自适应、可演进的智能 Agent 运行框架工厂

- [[harnessx|HarnessX（源摘要）]] — 9 维类型化组件 + AEGIS 演化引擎 + 框架-模型协同优化闭环，5 基准平均 +14.5%。
- [[harnessx-benchmark-evaluation|Benchmark 评测体系]] — 5 个基准适配器（GAIA/SWE-bench/TAU2/TB2/LoCoMo），15 个可复用 bench 模式。
- [[harnessx-framework-composition|框架组合机制]] — 8 钩子 + Processor 管道 + HarnessBuilder `|` 组合 + 13 个 Control 处理器。
- [[harnessx-aegis-evolution|AEGIS 演化引擎]] — Digester/Planner/Evolver/Critic + Seesaw 约束 + 跨框架 GRPO 协同演化。
- [[harnessx-sandbox-tools-rl|沙箱 / 工具 / RL 训练]] — ContextVar 沙箱注入 + 双轨 JSONL tracing + SGLangProvider token 级 RL 训练桥。

## 核心关注

1. **安全边界** — 代码执行、文件系统、网络调用的安全隔离
2. **资源管理** — Token 预算、并发控制、时间超限
3. **可观测性** — Agent 行为追踪、日志、调试
4. **扩展性** — Hook 系统、插件机制、自定义 Tool
5. **可靠性** — 错误恢复、幂等性、状态持久化

## 相关领域

- [[ai-coding/_index|AI Coding Best Practices]] — Agent 的编程能力
- [[rag-engineering/_index|RAG & Knowledge Engineering]] — Agent 的知识来源

## 标签

#agent #harness #tool-use #subagent #worktree #permissions

## Agent 论文地图

> [papers/](papers/) — 按 Agent 研究主题分类的论文笔记

### 综述与分类框架

> [papers/surveys/](papers/surveys/)

- [[survey-llm-based-autonomous-agents|LLM-based autonomous agents 综述]] — 按构建方法、应用场景和评测体系梳理 Agent 研究版图。
- [[rise-and-potential-llm-based-agents|Rise and Potential of LLM-based Agents]] — 用 brain / perception / action 框架组织 LLM Agent，适合作为 Agent 系统架构的顶层参考。
- [[survey-llm-based-multi-agents|多智能体综述]] — 覆盖角色、通信、组织结构、协作机制和应用场景。
- [[self-improvements-in-modern-agentic-systems|自提升智能体综述]] — 二元架构（模型+Harness）+ 两条正交优化路径 + AEGIS/GRPO + 三大失效模式，与 [[harnessx]] 互为理论/工程互补。

### 架构、规划、记忆与反思

> [papers/architecture/](papers/architecture/)

- [[react-reasoning-acting-language-models|ReAct]] — 将 reasoning 与 acting 交错，形成 Thought-Action-Observation 的经典 Agent 循环。
- [[reflexion|Reflexion]] — 通过语言反思和 episodic memory 改进后续尝试，是自我改进型 Agent 的代表。
- [[generative-agents-interactive-simulacra|Generative Agents]] — 用记忆流、反思和计划模拟可信的人类日常行为。
- [[tree-of-thoughts|Tree of Thoughts]] — 将单路径推理扩展为可搜索的 thought tree，适合复杂规划与回溯。

### 工具使用与 API 调用

> [papers/tool-use/](papers/tool-use/)

- [[toolformer|Toolformer]] — 通过自监督数据构造让模型学习何时以及如何调用工具。
- [[gorilla-large-language-model-connected-apis|Gorilla]] — 面向大规模 API 调用，强调通过检索和微调降低 API hallucination。
- [[toolllm|ToolLLM]] — 让模型学习真实 API 生态中的工具选择、多步调用和结果整合。

### Web、GUI 与 Computer Use

> [papers/web-gui/](papers/web-gui/)

- [[mind2web|Mind2Web]] — 用真实网站数据评估 Web Agent 的元素选择与跨网站泛化能力。
- [[webarena|WebArena]] — 构建可复现的真实风格网页环境，测试端到端多步网页任务完成能力。
- [[visualwebarena|VisualWebArena]] — 在 Web Agent 评测中加入视觉理解，覆盖截图、布局和非文本 UI 线索。
- [[osworld|OSWorld]] — 在真实操作系统环境中评估多模态 Agent 的开放式桌面任务能力。

### Coding Agent 与软件工程

> [papers/coding-agent/](papers/coding-agent/)

- [[swe-bench|SWE-bench]] — 用真实 GitHub issue 和测试评估模型是否能修复真实软件问题。
- [[swe-agent|SWE-agent]] — 强调 Agent-Computer Interface 对自动化软件工程表现的决定性影响。

### 多智能体协作

> [papers/multi-agent/](papers/multi-agent/)

- [[camel-communicative-agents|CAMEL]] — 用角色扮演和对话协作探索多 Agent 任务分工。
- [[autogen|AutoGen]] — 将多 Agent 对话、工具执行和 human-in-the-loop 组织为工程框架。

### Agent 训练与后训练

> [papers/training/](papers/training/)

- [[fireact|FireAct]] — 研究用 Agent 交互轨迹微调模型，提升 ReAct 风格任务表现。
- [[agenttuning|AgentTuning]] — 通过 AgentInstruct 数据提升模型的通用 Agent 能力。

### 评测、安全与鲁棒性

> [papers/evaluation-safety/](papers/evaluation-safety/)

- [[agentbench|AgentBench]] — 统一评估 LLM 在多种交互环境中作为 Agent 的表现。
- [[toolemu|ToolEmu]] — 用模拟工具沙盒识别工具型 Agent 的高风险行为。
- [[agentdojo|AgentDojo]] — 评估工具型 Agent 在动态环境中面对 prompt injection 的鲁棒性。
- [[agent-safetybench|Agent-SafetyBench]] — 系统评估 LLM Agent 的安全风险和行动能力带来的危害放大。
