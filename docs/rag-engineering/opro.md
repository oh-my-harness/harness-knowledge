---
title: "OPRO — LLM as Optimizer"
type: paper-summary
tags:
  - paper
  - llm-optimizer
  - prompt-optimization
  - opro
source: "assets/papers/2024-large-language-models-as-optimizers.pdf"
aliases:
  - "Large Language Models as Optimizers"
  - "Optimization by PROmpting"
---

# OPRO: Large Language Models as Optimizers

> **论文来源**：ICLR 2024 (Conference Paper)  
> **作者**：Chengrun Yang, Xuezhi Wang, Yifeng Lu, Hanxiao Liu, Quoc V. Le, Denny Zhou, Xinyun Chen (Google DeepMind)  
> **PDF 路径**：`assets/papers/2024-large-language-models-as-optimizers.pdf`

## 核心思想

提出 **OPRO (Optimization by PROmpting)**，一种利用大语言模型（LLM）作为优化器的简单有效方法。优化任务以自然语言描述，LLM 在每一步根据包含历史解及其得分的 meta-prompt 生成新解，新解经评估后加入 prompt 继续迭代。

## 关键方法

- **Meta-Prompt 设计**：包含两部分——
  1. **优化轨迹**：历史生成的解及其得分（按升序排列）
  2. **任务描述**：自然语言描述 + 若干训练集示例
- **解生成**：每步生成多个候选解，通过温度参数平衡探索与利用
- **三大组件**：Optimizer LLM（生成新解） → Scorer LLM（评估解质量） → Meta-Prompt（记录迭代轨迹）

## 主要实验结果

| 基准 | 人工设计指令 | OPRO 优化后 | 提升 |
|---|---|---|---|
| GSM8K (Zero-shot) | 71.8% (Let's think step by step) | **80.2%** (PaLM 2-L-IT) | +8% |
| BBH (23个任务) | 基准 | **超过50%** 的任务提升超5% | 显著 |

- **最佳发现指令**："Take a deep breath and work on this problem step-by-step." (GSM8K, 80.2%)
- **迁移性**：GSM8K 优化得到的指令可迁移至 MultiArith 和 AQuA

## 意义与贡献

1. 首次系统性展示 LLM 可作为通用优化器，无需梯度或传统优化算法
2. 自动 prompt 优化能力超越人类设计的指令
3. 为 NLP 领域的 prompt engineering 提供了自动化范式

## 关联笔记

- [[docs/rag-engineering/_index|RAG & Knowledge Engineering]]
- [[docs/ai-coding/_index|AI Coding Best Practices]]
- [[docs/agent-harness/_index|Agent Harness]]
- [[README|LLM Wiki 首页]]

## 延伸方向

- 降低对初始化的敏感性
- 更好地平衡探索与利用
- 利用错误案例提供更丰富的反馈
