---
title: "Generative Agents — Interactive Simulacra of Human Behavior"
type: paper-summary
tags:
  - paper
  - agent
  - memory
  - planning
  - social-simulation
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-generative-agents-interactive-simulacra.pdf"
aliases:
  - "Generative Agents"
---

# Generative Agents: Interactive Simulacra of Human Behavior

> **arXiv**：https://arxiv.org/abs/2304.03442  
> **PDF 路径**：`assets/papers/2023-generative-agents-interactive-simulacra.pdf`

## 核心思想

论文提出一种能模拟人类日常行为的生成式 Agent 架构。Agent 通过记忆流、反思和计划形成持续性的行为，而不是每轮独立生成响应。

## 关键机制

- **Memory Stream**：记录 Agent 的观察、事件和行为。
- **Reflection**：从大量记忆中抽象出更高层次的信念和经验。
- **Planning**：把长期目标分解为日程和具体行动。

## 贡献

1. 证明 LLM Agent 可以在沙盒环境中产生可信的社会行为。
2. 把记忆、反思、规划组合成可复用的 Agent 架构。
3. 展示多 Agent 社会模拟中的信息传播、协作和角色一致性。

## 对 Agent Harness 的启发

生产级 Agent 需要区分原始事件日志、检索记忆和抽象反思。只保存聊天历史不足以支撑长期行为一致性。

## 关联笔记

- [[reflexion]]
- [[survey-llm-based-multi-agents]]
- [[context-management]]

## 局限与待验证

- 评估偏主观，真实任务完成能力不是重点。
- 长期模拟的事实一致性、成本和隐私管理仍然困难。

