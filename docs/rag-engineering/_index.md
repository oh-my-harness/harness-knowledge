---
title: RAG & Knowledge Engineering
tags: [rag, knowledge-engineering, index]
created: 2026-06-10
updated: 2026-06-10
---

# 📚 RAG & Knowledge Engineering

> 构建 RAG 系统和知识库的核心知识

## 目录索引

- [RAG 架构](rag-architecture.md) — 检索增强生成架构设计
- [Chunking 策略](chunking-strategies.md) — 文档分块技术
- [Embedding 模型选型](embedding-models.md) — 向量化模型对比与选择
- [向量数据库](vector-databases.md) — 向量存储与检索
- [检索优化](retrieval-optimization.md) — 检索质量提升技术
- [知识图谱集成](knowledge-graph.md) — 结合图谱增强检索
- [知识库维护](wiki-maintenance.md) — 知识库生命周期管理

## 核心关注

1. **检索质量** — 召回率与精度的平衡
2. **分块艺术** — chunk size、chunk overlap、chunk hierarchy
3. **模型选型** — embedding 与 reranker 的配合
4. **端到端评估** — 从检索到生成的完整链路评测
5. **知识保鲜** — 知识库更新的自动化策略

## 相关领域

- [[ai-coding/_index|AI Coding Best Practices]] — AI 时代的知识管理
- [[agent-harness/_index|Agent Harness]] — 基于知识的 Agent 系统

## 标签

#rag #embedding #vector-database #knowledge-graph #chunking

## OPRO — LLM as Optimizer

- link: [[docs/rag-engineering/opro.md]]
- summary: ICLR 2024 论文：利用 LLM 作为优化器，通过自然语言 prompt 迭代寻找最优解；在 GSM8K 和 BBH 上显著超越人工设计指令。

## LightRAG — Graph-Enhanced RAG

- link: [[docs/rag-engineering/lightrag.md]]
- summary: EMNLP 2025 论文：引入图结构到 RAG 索引与检索，双层检索范式 + 增量更新，效率远超 GraphRAG。
