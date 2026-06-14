---
title: "The Rise and Potential of Large Language Model Based Agents"
type: paper-summary
tags:
  - paper
  - agent
  - survey
  - architecture
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-rise-and-potential-llm-based-agents.pdf"
aliases:
  - "The Rise and Potential of LLM-based Agents"
---

# The Rise and Potential of Large Language Model Based Agents

> **论文类型**：综述  
> **arXiv**：https://arxiv.org/abs/2309.07864  
> **PDF 路径**：`assets/papers/2023-rise-and-potential-llm-based-agents.pdf`

## 核心思想

论文用 **brain、perception、action** 三个部分组织 LLM Agent：LLM 是决策中枢，感知模块把外部环境转成模型可理解的状态，行动模块把模型决策落实为工具调用或环境操作。

## 关键框架

- **Brain**：推理、规划、记忆、知识、泛化。
- **Perception**：文本、图像、网页、传感器等输入。
- **Action**：工具调用、API、机器人动作、环境交互。

## 对 Agent Harness 的启发

Harness 设计中应明确区分“模型推理”和“外部执行”。模型输出动作意图，Harness 负责校验、路由、执行、记录和回滚。

## 关联笔记

- [[survey-llm-based-autonomous-agents]]
- [[toolllm]]
- [[osworld]]

## 局限与待验证

- 框架偏宏观，对具体 runtime 接口和状态管理细节没有深入展开。
- 多模态感知和真实动作执行的可靠性仍是开放问题。

