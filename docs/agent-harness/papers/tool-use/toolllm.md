---
title: "ToolLLM — Facilitating Large Language Models to Master 16000+ Real-world APIs"
type: paper-summary
tags:
  - paper
  - tool-use
  - api
  - agent
  - benchmark
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-toolllm.pdf"
aliases:
  - "ToolLLM"
  - "ToolBench"
---

# ToolLLM: Facilitating Large Language Models to Master 16000+ Real-world APIs

> **arXiv**：https://arxiv.org/abs/2307.16789  
> **PDF 路径**：`assets/papers/2023-toolllm.pdf`

## 核心思想

ToolLLM 构建 ToolBench，让模型学习在真实 API 生态中完成工具选择、参数填充、多步调用和结果整合。它从“会调用一个工具”推进到“能在大量工具中规划调用链”。

## 关键机制

- **ToolBench**：包含大量真实 API 和指令数据。
- **DFSDT**：基于深度优先搜索的决策树采样，用于生成多步工具调用轨迹。
- **ToolEval**：用自动评估方法判断工具调用任务的完成质量。

## 贡献

1. 系统化构建大规模工具调用数据。
2. 强调多步工具调用和工具链规划。
3. 为工具型 Agent 的训练和评测提供基础设施。

## 对 Agent Harness 的启发

Harness 的工具层需要支持工具发现、schema 暴露、调用轨迹记录、错误恢复和多步调用编排。

## 关联笔记

- [[toolformer]]
- [[gorilla-large-language-model-connected-apis]]
- [[tool-use-system]]

## 局限与待验证

- 自动生成轨迹可能包含不自然或低质量调用。
- API 副作用、权限和安全边界仍需额外机制。

