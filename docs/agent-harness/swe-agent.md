---
title: "SWE-agent — Agent-Computer Interfaces Enable Automated Software Engineering"
type: paper-summary
tags:
  - paper
  - coding-agent
  - software-engineering
  - agent-computer-interface
  - benchmark
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2024-swe-agent.pdf"
aliases:
  - "SWE-agent"
---

# SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering

> **arXiv**：https://arxiv.org/abs/2405.15793  
> **PDF 路径**：`assets/papers/2024-swe-agent.pdf`

## 核心思想

SWE-agent 的核心观点是：模型能力之外，Agent 与电脑交互的接口设计会显著影响软件工程任务表现。好的 Agent-Computer Interface 可以让模型更有效地浏览、编辑、测试和修复代码。

## 关键机制

- 面向代码任务设计专用命令和反馈格式。
- 让 Agent 更容易定位文件、查看上下文、编辑局部代码。
- 在 SWE-bench 上验证接口设计的收益。

## 贡献

1. 强调 ACI 是 Coding Agent 的关键变量。
2. 证明同一类模型在不同工具接口下表现差异明显。
3. 为代码 Agent runtime 设计提供了实证依据。

## 对 Agent Harness 的启发

Harness 不只是“给模型一个 shell”。工具输出应紧凑、结构化、可恢复，并避免把无关噪声塞进上下文。

## 关联笔记

- [[swe-bench]]
- [[tool-use-system]]
- [[worktree-management]]

## 局限与待验证

- 接口优化可能对特定 benchmark 有适配。
- 面对大型多语言仓库、长构建链和私有依赖仍需要更多机制。

