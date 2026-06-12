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
