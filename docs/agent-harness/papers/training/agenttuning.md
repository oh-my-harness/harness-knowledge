---
title: "AgentTuning — Enabling Generalized Agent Abilities for LLMs"
type: paper-summary
tags:
  - paper
  - agent
  - instruction-tuning
  - training
  - generalization
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-agenttuning.pdf"
aliases:
  - "AgentTuning"
  - "AgentInstruct"
---

# AgentTuning: Enabling Generalized Agent Abilities for LLMs

> **arXiv**：https://arxiv.org/abs/2310.12823  
> **PDF 路径**：`assets/papers/2023-agenttuning.pdf`

## 核心思想

AgentTuning 通过构建 AgentInstruct 数据集，对开源 LLM 进行指令微调，使其获得更通用的 Agent 能力，包括工具使用、环境交互和复杂任务规划。

## 关键机制

- 汇集多种 Agent 任务数据。
- 保留通用语言能力的同时增强 Agent 行为。
- 评估不同任务上的泛化能力。

## 贡献

1. 提出面向通用 Agent 能力的指令微调范式。
2. 证明 Agent 数据可以提升开源模型的行动能力。
3. 讨论 Agent tuning 与普通 instruction tuning 的差异。

## 对 Agent Harness 的启发

如果 Harness 能统一轨迹 schema，就能把不同任务、工具和环境中的经验汇总成训练数据。

## 关联笔记

- [[fireact]]
- [[toolllm]]
- [[agentbench]]

## 局限与待验证

- 泛化能力依赖训练任务覆盖面。
- 后训练可能影响模型原有对话能力，需要平衡。

