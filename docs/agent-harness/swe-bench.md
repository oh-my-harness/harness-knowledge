---
title: "SWE-bench — Can Language Models Resolve Real-World GitHub Issues?"
type: paper-summary
tags:
  - paper
  - coding-agent
  - software-engineering
  - benchmark
  - agent
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-swe-bench.pdf"
aliases:
  - "SWE-bench"
---

# SWE-bench: Can Language Models Resolve Real-World GitHub Issues?

> **arXiv**：https://arxiv.org/abs/2310.06770  
> **PDF 路径**：`assets/papers/2023-swe-bench.pdf`

## 核心思想

SWE-bench 用真实 GitHub issue 和对应代码仓库评估模型能否修复真实软件问题。它把代码智能从函数级补全推进到仓库级理解、修改和测试。

## 任务形式

- 输入：issue 描述、代码仓库。
- 输出：patch。
- 评估：运行项目测试，判断 patch 是否解决问题且不破坏已有行为。

## 贡献

1. 建立真实软件工程任务 benchmark。
2. 让“能否修复真实 issue”成为 Coding Agent 核心指标。
3. 揭示仓库导航、上下文选择和测试反馈的重要性。

## 对 Agent Harness 的启发

Coding Agent 需要文件搜索、编辑、测试执行、日志解析、patch 管理和回滚能力。模型本身只是其中一层。

## 关联笔记

- [[swe-agent]]
- [[docs/ai-coding/_index|AI Coding Best Practices]]
- [[worktree-management]]

## 局限与待验证

- 测试通过不一定等价于完整修复。
- 数据集任务分布偏 Python 开源项目，真实企业仓库还会有权限和环境问题。

