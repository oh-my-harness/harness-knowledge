---
title: "Agent-SafetyBench — Evaluating the Safety of LLM Agents"
type: paper-summary
tags:
  - paper
  - agent
  - safety
  - benchmark
  - evaluation
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2024-agent-safetybench.pdf"
aliases:
  - "Agent-SafetyBench"
---

# Agent-SafetyBench: Evaluating the Safety of LLM Agents

> **arXiv**：https://arxiv.org/abs/2412.14470  
> **PDF 路径**：`assets/papers/2024-agent-safetybench.pdf`

## 核心思想

Agent-SafetyBench 系统评估 LLM Agent 的安全风险，重点关注 Agent 在具备工具调用和环境交互能力后，可能出现的越权、误操作、欺骗、隐私和高风险执行问题。

## 关注维度

- 工具调用安全
- 隐私与敏感信息泄露
- 越权执行
- 有害任务协助
- 长程任务中的目标偏移

## 贡献

1. 从 Agent 角度组织安全评测，而不是只看普通文本安全。
2. 强调行动能力会放大模型错误的现实后果。
3. 为 Agent 安全基准和防御策略提供参考。

## 对 Agent Harness 的启发

安全策略应进入 runtime：权限模型、工具分级、人工确认、审计日志、最小权限、敏感数据隔离和失败回滚都应成为默认机制。

## 关联笔记

- [[agentdojo]]
- [[toolemu]]
- [[permission-model]]

## 局限与待验证

- 安全 benchmark 很难穷尽真实风险。
- 不同应用域的安全边界差异大，需要领域化策略。

