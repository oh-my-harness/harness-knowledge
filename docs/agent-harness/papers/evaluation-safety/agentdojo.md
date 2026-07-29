---
title: "AgentDojo — A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents"
type: paper-summary
tags:
  - paper
  - agent
  - safety
  - prompt-injection
  - tool-use
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2024-agentdojo.pdf"
aliases:
  - "AgentDojo"
---

# AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents

> **arXiv**：https://arxiv.org/abs/2406.13352  
> **PDF 路径**：`assets/papers/2024-agentdojo.pdf`

## 核心思想

AgentDojo 评估工具型 Agent 面对 prompt injection 时的鲁棒性。它把用户任务、工具数据和恶意指令放在同一个动态环境中，测试 Agent 是否会被外部内容劫持。

## 关键问题

- Agent 能否区分用户指令和不可信外部内容？
- 工具返回中的恶意文本是否会改变 Agent 行为？
- 防御方法会不会降低正常任务完成率？

## 贡献

1. 把 prompt injection 从聊天安全问题扩展到工具型 Agent 场景。
2. 同时评估任务完成率和攻击成功率。
3. 提供动态环境来测试攻击与防御。

## 对 Agent Harness 的启发

Harness 必须标注信息来源和信任边界。来自网页、邮件、文档、工具返回的内容不能和用户/系统指令混在同一权限层。

## 关联笔记

- [[toolemu]]
- [[agent-safetybench]]
- [[permission-model]]

## 局限与待验证

- 攻击模式会持续演化，benchmark 需要不断更新。
- 过强防御可能降低 Agent 的正常可用性。

