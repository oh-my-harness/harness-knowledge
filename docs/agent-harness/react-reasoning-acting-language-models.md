---
title: "ReAct — Synergizing Reasoning and Acting in Language Models"
type: paper-summary
tags:
  - paper
  - agent
  - reasoning
  - tool-use
  - planning
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2022-react-reasoning-acting-language-models.pdf"
aliases:
  - "ReAct"
---

# ReAct: Synergizing Reasoning and Acting in Language Models

> **arXiv**：https://arxiv.org/abs/2210.03629  
> **PDF 路径**：`assets/papers/2022-react-reasoning-acting-language-models.pdf`

## 核心思想

ReAct 提出把 **Reasoning** 和 **Acting** 交错在同一条轨迹里：模型先产生思考，再选择动作，观察环境返回，再继续思考和行动。它奠定了很多现代 LLM Agent 的基本循环。

## 方法结构

```text
Thought -> Action -> Observation -> Thought -> Action -> ...
```

- **Thought**：解释当前状态、分解任务、修正策略。
- **Action**：搜索、查表、调用工具、操作环境。
- **Observation**：外部工具或环境返回的结果。

## 贡献

1. 让语言模型不再只做静态问答，而是可以通过行动获取新信息。
2. 推理轨迹提升可解释性，便于调试 Agent 行为。
3. 在知识问答和交互式决策任务中证明了 reasoning-action loop 的价值。

## 对 Agent Harness 的启发

Harness 的最小内核可以围绕 ReAct loop 设计：解析动作、执行工具、收集 observation、追加到上下文、进入下一轮。

## 关联笔记

- [[tool-use-system]]
- [[reflexion]]
- [[agentbench]]

## 局限与待验证

- 轨迹依赖 prompt，稳定性和格式遵循能力受模型影响。
- 长任务中上下文膨胀明显，需要记忆压缩和状态管理。

