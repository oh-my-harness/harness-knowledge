---
title: "OSWorld — Benchmarking Multimodal Agents for Open-Ended Tasks in Real Computer Environments"
type: paper-summary
tags:
  - paper
  - gui-agent
  - computer-use
  - multimodal
  - benchmark
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2024-osworld.pdf"
aliases:
  - "OSWorld"
---

# OSWorld: Benchmarking Multimodal Agents for Open-Ended Tasks in Real Computer Environments

> **arXiv**：https://arxiv.org/abs/2404.07972  
> **PDF 路径**：`assets/papers/2024-osworld.pdf`

## 核心思想

OSWorld 评估多模态 Agent 在真实操作系统中完成开放式任务的能力。任务不局限于浏览器，而是覆盖桌面应用、文件系统、办公软件和系统设置。

## 关键特点

- 真实 OS 环境中的多步任务。
- 需要截图理解、鼠标键盘控制、应用切换。
- 评估任务完成而不是单步动作正确率。

## 贡献

1. 将 Agent 评测推进到完整 computer-use 场景。
2. 暴露 GUI grounding、状态恢复和长程执行的困难。
3. 为通用桌面 Agent 提供了重要 benchmark。

## 对 Agent Harness 的启发

Computer-use Harness 需要隔离环境、可重置镜像、屏幕录制、输入控制、状态检查和安全权限边界。

## 关联笔记

- [[visualwebarena]]
- [[agentbench]]
- permission-model

## 局限与待验证

- 真实 GUI 环境复现成本高。
- 任务判定和异常恢复比网页环境更复杂。

