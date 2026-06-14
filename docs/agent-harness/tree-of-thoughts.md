---
title: "Tree of Thoughts — Deliberate Problem Solving with Large Language Models"
type: paper-summary
tags:
  - paper
  - reasoning
  - planning
  - search
  - agent
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-tree-of-thoughts.pdf"
aliases:
  - "Tree of Thoughts"
  - "ToT"
---

# Tree of Thoughts: Deliberate Problem Solving with Large Language Models

> **arXiv**：https://arxiv.org/abs/2305.10601  
> **PDF 路径**：`assets/papers/2023-tree-of-thoughts.pdf`

## 核心思想

Tree of Thoughts 把推理过程从单条链扩展为搜索树。模型可以生成多个中间想法，对它们打分，再通过 BFS、DFS 等搜索策略选择更有希望的路径。

## 方法结构

- **Thought generation**：生成多个候选中间步骤。
- **State evaluation**：评估某个中间状态是否有前景。
- **Search algorithm**：在候选 thought tree 中探索、回溯和选择。

## 贡献

1. 让 LLM 推理从一次性生成变成可控搜索。
2. 适合需要试探、回溯和全局规划的问题。
3. 给 Agent planner 提供了可组合的搜索式思路。

## 对 Agent Harness 的启发

复杂 Agent 任务可以把“下一步动作”扩展为候选动作树，再结合成本、风险和成功概率选择执行路径。

## 关联笔记

- [[react-reasoning-acting-language-models]]
- [[reflexion]]
- [[agentbench]]

## 局限与待验证

- 计算成本显著高于单路径推理。
- 状态评估本身仍依赖 LLM，可能出现自信但错误的路径选择。

