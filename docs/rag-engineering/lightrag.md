---
title: "LightRAG — Graph-Enhanced Retrieval-Augmented Generation"
type: paper-summary
tags:
  - paper
  - rag
  - knowledge-graph
  - retrieval
source: "assets/papers/lightrag-simple-fast-retrieval-augmented-generation.pdf"
aliases:
  - "LightRAG"
  - "Graph-Powered RAG"
---

# LightRAG: Simple and Fast Retrieval-Augmented Generation

> **会议**：EMNLP 2025 (Findings)  
> **作者**：Zirui Guo, Lianghao Xia, Yanhua Yu, Tu Ao, Chao Huang (University of Hong Kong & BUPT)  
> **PDF 路径**：`assets/papers/lightrag-simple-fast-retrieval-augmented-generation.pdf`

## 核心思想

LightRAG 将**图结构**引入 RAG 系统的文本索引和检索过程，通过**双层检索范式**（低层具体实体 + 高层抽象主题）和**增量更新算法**，实现高效、全面、可扩展的检索增强生成。

## 关键方法

### 1. 基于图的文本索引
- **实体与关系提取**：LLM 从分块文档中提取实体（节点）和关系（边）
- **LLM 概要分析 (Profiling)**：为每个节点/边生成键值对（key=简短名称，value=文本摘要）
- **去重优化**：合并相同实体/关系，降低图操作开销

### 2. 双层检索范式

| 层次 | 对象 | 示例查询 |
|---|---|---|
| **低层 (Low-Level)** | 具体实体及其属性/关系 | "谁写了《傲慢与偏见》？" |
| **高层 (High-Level)** | 广泛主题和概括性概念 | "人工智能如何影响现代教育？" |

- 同时利用**图结构 + 向量表示**：提取局部和全局关键词 → 向量匹配 → 融合一跳邻居信息

### 3. 增量更新
- 新文档直接提取实体/关系并合并到现有图，**无需重建整个索引**
- 大幅降低计算开销和适应时间

## 实验结果 (Win Rate vs 基线平均)

| 基线 | 综合 | 多样性 | 赋能 | 总体 |
|---|---|---|---|---|
| vs NaiveRAG | **74%** | **73%** | **73%** | **73%** |
| vs GraphRAG | **52%** | **69%** | **55%** | **52%** |

- 在最大数据集 (Legal, 5M tokens) 上优势最明显
- 检索阶段仅需 **<100 tokens**（vs GraphRAG 的 610K tokens）
- 响应速度：**11.2s**（vs GraphRAG 的 23.6s）

## 意义与贡献

1. 首次将图结构系统性地融入 RAG 全文索引流程
2. 双层检索兼顾具体细节与全局主题
3. 增量更新机制使系统能高效应对动态数据环境
4. 显著优于 GraphRAG 的效率—轻量、快速、低成本

## 关联笔记

- [[docs/rag-engineering/_index|RAG & Knowledge Engineering]]
- [[README|LLM Wiki 首页]]
- [[docs/agent-harness/_index|Agent Harness]]

## 延伸方向

- 多模态扩展（文本 + 图像 + 音频）
- 时序感知（反映动态事件与上下文演进）
