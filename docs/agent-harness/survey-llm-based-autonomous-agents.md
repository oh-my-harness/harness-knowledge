---
title: "A Survey on Large Language Model based Autonomous Agents"
type: paper-summary
tags:
  - paper
  - agent
  - survey
  - autonomous-agents
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-survey-llm-based-autonomous-agents.pdf"
aliases:
  - "LLM-based Autonomous Agents Survey"
---

# A Survey on Large Language Model based Autonomous Agents

> **论文类型**：综述  
> **arXiv**：https://arxiv.org/abs/2308.11432  
> **PDF 路径**：`assets/papers/2023-survey-llm-based-autonomous-agents.pdf`

## 核心思想

这篇综述把 LLM-based Agent 视为由大语言模型驱动、能自主感知、规划、行动和学习的系统。它的价值在于给 Agent 研究建立了一张比较完整的地图：Agent 如何构建、可以用于哪些场景、又应该如何评估。

## 关键框架

- **Agent 构建**：包括 profile、memory、planning、action 等模块。
- **Agent 应用**：覆盖社会模拟、软件工程、科学研究、游戏、机器人等场景。
- **Agent 评估**：关注任务完成率、交互能力、泛化能力、安全性和可信性。

## 对 Agent Harness 的启发

Agent Harness 可以把这篇论文当作顶层 taxonomy：运行时需要承载 profile、memory、planner、tool/action executor、feedback loop 等模块，而不是只把 Agent 理解为一段 prompt。

## 关联笔记

- [[docs/agent-harness/_index|Agent Harness]]
- [[react-reasoning-acting-language-models]]
- [[reflexion]]
- [[agentbench]]

## 局限与待验证

- 综述覆盖广，但对不同 benchmark 的可比性讨论有限。
- 对生产级 Agent 的权限、审计、恢复、成本控制等工程问题展开不足。

