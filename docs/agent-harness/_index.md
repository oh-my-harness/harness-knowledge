---
title: Agent Harness
tags: [agent-harness, harness, index]
created: 2026-06-10
updated: 2026-06-10
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

## DeepTutor — Agentic Personalized Tutoring

- link: [[docs/agent-harness/deeptutor.md]]
- summary: HKU 开源智能体化辅导框架：混合个性化引擎 + 闭环任务辅导，TutorBench 评估 +10.76%，通用推理 +29.4%。

## Reflexion — Verbal Reinforcement Learning

- link: [[docs/agent-harness/reflexion.md]]
- summary: 语言智能体通过自我反思进行强化学习：HumanEval 91% Pass@1 SOTA，ALFWorld 97% 成功率，无需微调。

## FluxEDA — Stateful Agentic EDA Infrastructure

- link: [[docs/agent-harness/fluxeda.md]]
- summary: 浙江大学：统一有状态基础设施，使 LLM 智能体在商业 EDA 工具上执行多步迭代优化，支持状态回滚和分支探索。

## Agent 论文地图

### 综述与分类框架

- link: [[docs/agent-harness/survey-llm-based-autonomous-agents.md]]
- summary: LLM-based autonomous agents 综述，按构建方法、应用场景和评测体系梳理 Agent 研究版图。

- link: [[docs/agent-harness/rise-and-potential-llm-based-agents.md]]
- summary: 用 brain / perception / action 框架组织 LLM Agent，适合作为 Agent 系统架构的顶层参考。

- link: [[docs/agent-harness/survey-llm-based-multi-agents.md]]
- summary: 多智能体综述，覆盖角色、通信、组织结构、协作机制和应用场景。

### 架构、规划、记忆与反思

- link: [[docs/agent-harness/react-reasoning-acting-language-models.md]]
- summary: ReAct 将 reasoning 与 acting 交错，形成 Thought-Action-Observation 的经典 Agent 循环。

- link: [[docs/agent-harness/reflexion.md]]
- summary: Reflexion 通过语言反思和 episodic memory 改进后续尝试，是自我改进型 Agent 的代表。

- link: [[docs/agent-harness/generative-agents-interactive-simulacra.md]]
- summary: Generative Agents 用记忆流、反思和计划模拟可信的人类日常行为。

- link: [[docs/agent-harness/tree-of-thoughts.md]]
- summary: Tree of Thoughts 将单路径推理扩展为可搜索的 thought tree，适合复杂规划与回溯。

### 工具使用与 API 调用

- link: [[docs/agent-harness/toolformer.md]]
- summary: Toolformer 通过自监督数据构造让模型学习何时以及如何调用工具。

- link: [[docs/agent-harness/gorilla-large-language-model-connected-apis.md]]
- summary: Gorilla 面向大规模 API 调用，强调通过检索和微调降低 API hallucination。

- link: [[docs/agent-harness/toolllm.md]]
- summary: ToolLLM / ToolBench 让模型学习真实 API 生态中的工具选择、多步调用和结果整合。

### Web、GUI 与 Computer Use

- link: [[docs/agent-harness/mind2web.md]]
- summary: Mind2Web 用真实网站数据评估 Web Agent 的元素选择与跨网站泛化能力。

- link: [[docs/agent-harness/webarena.md]]
- summary: WebArena 构建可复现的真实风格网页环境，测试端到端多步网页任务完成能力。

- link: [[docs/agent-harness/visualwebarena.md]]
- summary: VisualWebArena 在 Web Agent 评测中加入视觉理解，覆盖截图、布局和非文本 UI 线索。

- link: [[docs/agent-harness/osworld.md]]
- summary: OSWorld 在真实操作系统环境中评估多模态 Agent 的开放式桌面任务能力。

### Coding Agent 与软件工程

- link: [[docs/agent-harness/swe-bench.md]]
- summary: SWE-bench 用真实 GitHub issue 和测试评估模型是否能修复真实软件问题。

- link: [[docs/agent-harness/swe-agent.md]]
- summary: SWE-agent 强调 Agent-Computer Interface 对自动化软件工程表现的决定性影响。

### 多智能体协作

- link: [[docs/agent-harness/camel-communicative-agents.md]]
- summary: CAMEL 用角色扮演和对话协作探索多 Agent 任务分工。

- link: [[docs/agent-harness/autogen.md]]
- summary: AutoGen 将多 Agent 对话、工具执行和 human-in-the-loop 组织为工程框架。

### Agent 训练与后训练

- link: [[docs/agent-harness/fireact.md]]
- summary: FireAct 研究用 Agent 交互轨迹微调模型，提升 ReAct 风格任务表现。

- link: [[docs/agent-harness/agenttuning.md]]
- summary: AgentTuning 通过 AgentInstruct 数据提升模型的通用 Agent 能力。

### 评测、安全与鲁棒性

- link: [[docs/agent-harness/agentbench.md]]
- summary: AgentBench 统一评估 LLM 在多种交互环境中作为 Agent 的表现。

- link: [[docs/agent-harness/toolemu.md]]
- summary: ToolEmu 用模拟工具沙盒识别工具型 Agent 的高风险行为。

- link: [[docs/agent-harness/agentdojo.md]]
- summary: AgentDojo 评估工具型 Agent 在动态环境中面对 prompt injection 的鲁棒性。

- link: [[docs/agent-harness/agent-safetybench.md]]
- summary: Agent-SafetyBench 系统评估 LLM Agent 的安全风险和行动能力带来的危害放大。
