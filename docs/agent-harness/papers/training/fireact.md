---
title: "FireAct — Toward Language Agent Fine-tuning"
type: paper-summary
tags:
  - paper
  - agent
  - fine-tuning
  - tool-use
  - training
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-fireact.pdf"
aliases:
  - "FireAct"
---

# FireAct: Toward Language Agent Fine-tuning

> **arXiv**：https://arxiv.org/abs/2310.05915  
> **PDF 路径**：`assets/papers/2023-fireact.pdf`

## 核心思想

FireAct 研究如何用 Agent 交互轨迹微调语言模型，使模型更稳定地执行 ReAct 风格的推理和行动。它把 Agent 能力从 prompt engineering 推向数据驱动训练。

## 关键问题

- 哪些轨迹适合用于 Agent fine-tuning？
- 小模型能否通过轨迹学习获得较强 Agent 能力？
- 训练后的模型是否能在新任务上泛化？

## 贡献

1. 证明 Agent 轨迹数据可以提升工具使用和多步任务表现。
2. 强调高质量交互数据比单纯增加 prompt 技巧更可持续。
3. 为后续 Agent 后训练提供实验基础。

## 对 Agent Harness 的启发

Harness 应记录可复用轨迹：任务、思考、动作、观察、错误、最终结果。这些日志未来可以变成训练和评测资产。

## 关联笔记

- [[react-reasoning-acting-language-models]]
- [[agenttuning]]
- tool-use-system

## 局限与待验证

- 轨迹质量决定训练上限，低质量轨迹可能固化错误策略。
- 训练收益与模型规模、任务类型和工具环境强相关。

