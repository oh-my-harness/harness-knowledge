---
title: "AutoGen — Enabling Next-Gen LLM Applications via Multi-Agent Conversation"
type: paper-summary
tags:
  - paper
  - multi-agent
  - framework
  - conversation
  - agent
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-autogen.pdf"
aliases:
  - "AutoGen"
---

# AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation

> **arXiv**：https://arxiv.org/abs/2308.08155  
> **PDF 路径**：`assets/papers/2023-autogen.pdf`

## 核心思想

AutoGen 是一个面向多 Agent 对话应用的框架，通过可配置 Agent、对话模式和工具执行机制，把复杂 LLM 应用组织为多个 Agent 之间的交互。

## 关键机制

- **Conversable Agent**：统一人类、LLM、工具执行器等参与方。
- **Conversation programming**：用对话流组织任务执行。
- **Human-in-the-loop**：支持人在关键步骤介入。

## 贡献

1. 将多 Agent 从论文原型推进到工程框架。
2. 支持不同角色、工具和人工反馈混合编排。
3. 影响了后续大量 Agent 应用框架设计。

## 对 Agent Harness 的启发

多 Agent Harness 应把消息、角色、工具执行和人工确认都视为一等对象，并提供可观测的运行轨迹。

## 关联笔记

- [[camel-communicative-agents]]
- [[survey-llm-based-multi-agents]]
- [[subagent-dispatch]]

## 局限与待验证

- 对话式编排易产生长上下文和成本问题。
- 多 Agent 任务质量高度依赖流程设计。

