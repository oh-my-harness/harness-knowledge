---
title: "AgentBench — Evaluating LLMs as Agents"
type: paper-summary
tags:
  - paper
  - agent
  - benchmark
  - evaluation
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-agentbench.pdf"
aliases:
  - "AgentBench"
---

# AgentBench: Evaluating LLMs as Agents

> **arXiv**：https://arxiv.org/abs/2308.03688  
> **PDF 路径**：`assets/papers/2023-agentbench.pdf`

## 核心思想

AgentBench 旨在系统评估 LLM 作为 Agent 在多种交互环境中的表现。它关注模型是否能在环境反馈中持续决策，而不是只回答静态问题。

## 评测环境

- 操作系统/命令行任务
- 数据库任务
- 网页购物
- 游戏和交互式决策
- 知识问答与推理环境

## 贡献

1. 将多环境 Agent 能力纳入统一评测。
2. 体现交互、规划、工具使用和环境反馈的重要性。
3. 为比较不同 LLM 的 Agent 能力提供基准。

## 对 Agent Harness 的启发

评测 Harness 需要环境适配层、动作规范、状态观测、成功判定和日志记录。不同环境的统一抽象很重要。

## 关联笔记

- [[react-reasoning-acting-language-models]]
- [[webarena]]
- [[osworld]]

## 局限与待验证

- 多环境统一评测会牺牲部分领域深度。
- 指标很难完全反映真实生产任务的可靠性。

