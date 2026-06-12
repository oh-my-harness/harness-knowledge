---
title: "Reflexion — Language Agents with Verbal Reinforcement Learning"
type: paper-summary
tags:
  - paper
  - agent
  - reinforcement-learning
  - self-reflection
  - code-generation
source: "assets/papers/reflexion-language-agents-verbal-reinforcement-learning.pdf"
aliases:
  - "Reflexion"
  - "Verbal Reinforcement Learning"
---

# Reflexion: Language Agents with Verbal Reinforcement Learning

> **作者**：Noah Shinn, Federico Cassano, Edward Berman, Ashwin Gopinath, Karthik Narasimhan, Shunyu Yao (Northeastern University, MIT, Princeton University)  
> **PDF 路径**：`assets/papers/reflexion-language-agents-verbal-reinforcement-learning.pdf`

## 核心思想

Reflexion 提出一种**语言强化学习（Verbal Reinforcement Learning）** 新范式——不通过权重更新，而是通过**语言反馈**来强化智能体行为。智能体对自己的失败进行口头反思（Self-Reflection），将经验存储到情景记忆（Episodic Memory）中，在下一次尝试中利用这些经验改进决策。

## 架构：三个模型协同

```
Actor (Ma)  →  生成文本/动作
Evaluator (Me)  →  评估输出质量（标量奖励）
Self-Reflection (Msr)  →  生成口头强化反馈（存储到记忆）
```

- **短期记忆**：当前轨迹历史
- **长期记忆**：自反思输出的经验摘要（一般保留最近 1-3 条）

## 主要实验结果

| 任务 | 基线 | Reflexion | 提升 |
|---|---|---|---|
| **ALFWorld** (决策) | ReAct 78% | **130/134 (97%)** | **+22%** |
| **HotPotQA** (推理) | CoT + ReAct | 各提升 14-20% | 显著 |
| **HumanEval** (Python 编程) | GPT-4 80% | **91% Pass@1** | **+11%** |

### 代码生成 SOTA 表现

| 基准 | 语言 | 前 SOTA | Reflexion |
|---|---|---|---|
| HumanEval | Python | 80.1% (GPT-4) | **91.0%** |
| HumanEval | Rust | 60.0% (GPT-4) | **68.0%** |
| LeetcodeHard | Python | 7.5% (GPT-4) | **15.0%** |

## 关键技术细节

- **自评估机制**：
  - 决策任务：启发式规则（重复动作检测）+ LLM 分类
  - 编程任务：自生成单元测试套件（最多6个）→ AST 过滤 → 执行验证
- **记忆容量**：滑动窗口最多保留 3 条反思经验
- **消融实验证明**：仅有试错 + 测试执行（无自反思）无法提升性能

## 意义与贡献

1. 首次提出"语言强化学习"概念，用自然语言替代梯度做策略优化
2. 提出的 LeetcodeHardGym（40 道 LeetCode Hard 题目）成为新的代码生成 RL 基准
3. 证明自反思能力是强大 LLM 的涌现特性，且能显著提升智能体学习效率
4. 在决策、推理、编程三类任务上均取得显著提升

## 关联笔记

- [[docs/agent-harness/_index|Agent Harness]]
- [[docs/ai-coding/_index|AI Coding Best Practices]]
- [[docs/rag-engineering/_index|RAG & Knowledge Engineering]]
- [[README|LLM Wiki 首页]]

## 局限性

- 可能陷入局部最优（如 WebShop 电商任务中表现不佳）
- 依赖 LLM 的自我评估能力（较弱模型无法有效自反思）
- 长轨迹中早期错误的追溯仍具挑战
