---
title: "Gorilla — Large Language Model Connected with Massive APIs"
type: paper-summary
tags:
  - paper
  - tool-use
  - api
  - agent
  - retrieval
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-gorilla-large-language-model-connected-apis.pdf"
aliases:
  - "Gorilla"
---

# Gorilla: Large Language Model Connected with Massive APIs

> **arXiv**：https://arxiv.org/abs/2305.15334  
> **PDF 路径**：`assets/papers/2023-gorilla-large-language-model-connected-apis.pdf`

## 核心思想

Gorilla 关注大规模 API 调用场景，目标是让 LLM 根据用户需求生成正确的 API 调用，并降低 API hallucination。它结合指令微调与检索增强，使模型能面对大量不断变化的 API 文档。

## 关键机制

- **APIBench**：覆盖 TorchHub、TensorHub、HuggingFace 等 API。
- **Instruction tuning**：训练模型生成符合文档的 API 调用。
- **Retrieval-aware inference**：检索相关 API 文档，降低过时和幻觉问题。

## 贡献

1. 把工具使用问题推进到大规模 API 生态。
2. 明确提出 API hallucination 是工具型 Agent 的关键风险。
3. 展示 RAG 与工具调用训练结合的价值。

## 对 Agent Harness 的启发

工具注册表需要可检索、可版本化、可校验。Agent 不能只凭模型内部记忆调用 API。

## 关联笔记

- [[toolformer]]
- [[toolllm]]
- [[docs/rag-engineering/_index|RAG & Knowledge Engineering]]

## 局限与待验证

- 重点在生成 API 调用，不等同于完整长程任务执行。
- 真实 API 的权限、速率限制、失败恢复和副作用控制仍需 Harness 处理。

