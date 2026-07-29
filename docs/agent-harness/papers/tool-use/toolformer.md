---
title: "Toolformer — Language Models Can Teach Themselves to Use Tools"
type: paper-summary
tags:
  - paper
  - tool-use
  - api
  - agent
  - self-supervised-learning
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-toolformer.pdf"
aliases:
  - "Toolformer"
---

# Toolformer: Language Models Can Teach Themselves to Use Tools

> **arXiv**：https://arxiv.org/abs/2302.04761  
> **PDF 路径**：`assets/papers/2023-toolformer.pdf`

## 核心思想

Toolformer 研究语言模型如何通过自监督方式学习调用外部工具。模型先在文本中自动标注可能的 API 调用，再保留能降低语言建模损失的调用样本，用这些数据继续训练。

## 关键工具类型

- 计算器
- 问答系统
- 搜索引擎
- 翻译系统
- 日历/时间工具

## 贡献

1. 把工具调用学习转化为数据自动构造和语言建模问题。
2. 证明模型可以学习“何时调用工具”和“如何插入调用结果”。
3. 为后续 Tool-use Agent 和函数调用训练提供了重要思路。

## 对 Agent Harness 的启发

工具调用能力不只来自 runtime，也依赖模型训练数据。Harness 应提供稳定 schema、返回格式和错误反馈，便于未来沉淀调用轨迹。

## 关联笔记

- tool-use-system
- [[toolllm]]
- [[gorilla-large-language-model-connected-apis]]

## 局限与待验证

- 工具集合相对简单，离真实复杂 API 生态还有距离。
- 自监督筛选依赖语言建模损失，不一定等价于真实任务成功率。

