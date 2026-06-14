---
title: "Large Language Model based Multi-Agents: A Survey"
type: paper-summary
tags:
  - paper
  - agent
  - multi-agent
  - survey
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2024-survey-llm-based-multi-agents.pdf"
aliases:
  - "LLM-based Multi-Agents Survey"
---

# Large Language Model based Multi-Agents: A Survey

> **论文类型**：综述  
> **arXiv**：https://arxiv.org/abs/2402.01680  
> **PDF 路径**：`assets/papers/2024-survey-llm-based-multi-agents.pdf`

## 核心思想

这篇综述聚焦多智能体系统，讨论多个 LLM Agent 如何通过角色分工、通信、协作、竞争或社会模拟完成复杂任务。

## 关键框架

- **Agent 构成**：角色、能力、记忆、目标。
- **交互方式**：对话、辩论、投票、协作规划、任务委派。
- **组织结构**：扁平协作、层级组织、专家委员会、模拟社会。
- **应用场景**：软件开发、社会模拟、游戏、教育、科学发现。

## 对 Agent Harness 的启发

多 Agent 不只是“多开几个模型”，而是需要显式的调度协议、共享状态、任务边界、停止条件和冲突解决机制。

## 关联笔记

- [[camel-communicative-agents]]
- [[autogen]]
- [[subagent-dispatch]]

## 局限与待验证

- 多 Agent 的收益常被成本、冗余沟通和错误扩散抵消。
- 真实业务中需要更强的可观测性与责任归因。

