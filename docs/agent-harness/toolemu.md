---
title: "ToolEmu — Identifying the Risks of LM Agents with an Emulated Sandbox"
type: paper-summary
tags:
  - paper
  - agent
  - safety
  - tool-use
  - evaluation
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-toolemu.pdf"
aliases:
  - "ToolEmu"
---

# ToolEmu: Identifying the Risks of LM Agents with an Emulated Sandbox

> **arXiv**：https://arxiv.org/abs/2309.15817  
> **PDF 路径**：`assets/papers/2023-toolemu.pdf`

## 核心思想

ToolEmu 用模拟工具环境评估工具型语言 Agent 的风险。它避免直接在真实世界执行危险动作，同时让 Agent 面对接近真实的工具调用后果。

## 风险类型

- 错误调用高影响工具。
- 忽略用户约束或安全要求。
- 误解工具返回结果。
- 在不确定时仍执行不可逆操作。

## 贡献

1. 提供一种安全评估工具型 Agent 的模拟方法。
2. 强调 Agent 风险来自“语言输出 + 外部动作”的组合。
3. 帮助发现普通静态问答评测无法暴露的问题。

## 对 Agent Harness 的启发

高风险工具应先进入 sandbox 或 dry-run 模式，并提供显式确认、权限分级和可审计日志。

## 关联笔记

- [[agentdojo]]
- [[agent-safetybench]]
- [[permission-model]]

## 局限与待验证

- 模拟环境与真实工具仍存在差距。
- 风险分类和判定标准需要结合具体业务场景定制。

