---
title: "CAMEL — Communicative Agents for Mind Exploration of Large Scale Language Model Society"
type: paper-summary
tags:
  - paper
  - multi-agent
  - role-playing
  - collaboration
  - agent
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-camel-communicative-agents.pdf"
aliases:
  - "CAMEL"
---

# CAMEL: Communicative Agents for Mind Exploration of Large Scale Language Model Society

> **arXiv**：https://arxiv.org/abs/2303.17760  
> **PDF 路径**：`assets/papers/2023-camel-communicative-agents.pdf`

## 核心思想

CAMEL 用角色扮演的方式让多个语言 Agent 通过对话协作完成任务。它把任务拆成不同角色，例如 AI assistant 和 AI user，由双方持续互动推进解决方案。

## 关键机制

- **Role-playing**：给 Agent 分配明确角色和目标。
- **Inception prompting**：初始化角色、任务和沟通约束。
- **Conversational collaboration**：通过多轮对话推进任务。

## 贡献

1. 早期系统化探索 LLM 多 Agent 协作。
2. 提出角色设定对任务分工和行为稳定性的影响。
3. 促进后续多 Agent 框架和社会模拟研究。

## 对 Agent Harness 的启发

多 Agent 需要明确角色、协议、消息格式和终止条件，否则很容易变成成本高但信息增量低的对话。

## 关联笔记

- [[autogen]]
- [[survey-llm-based-multi-agents]]
- subagent-dispatch

## 局限与待验证

- 对话协作不一定带来真实任务性能提升。
- 容易产生角色漂移、重复讨论和错误共识。

