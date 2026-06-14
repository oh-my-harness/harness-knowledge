---
title: "VisualWebArena — Evaluating Multimodal Agents on Realistic Visual Web Tasks"
type: paper-summary
tags:
  - paper
  - web-agent
  - multimodal
  - benchmark
  - agent
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2024-visualwebarena.pdf"
aliases:
  - "VisualWebArena"
---

# VisualWebArena: Evaluating Multimodal Agents on Realistic Visual Web Tasks

> **arXiv**：https://arxiv.org/abs/2401.13649  
> **PDF 路径**：`assets/papers/2024-visualwebarena.pdf`

## 核心思想

VisualWebArena 在 WebArena 的基础上加入视觉信息，评估多模态 Agent 是否能理解网页截图、图片内容、视觉布局和非文本 UI 线索。

## 关键特点

- 包含需要视觉理解的网页任务。
- 结合截图、文本和结构化网页状态。
- 测试 Agent 对布局、图像、图标和视觉差异的理解能力。

## 贡献

1. 说明 Web Agent 不能只依赖 DOM 文本。
2. 推动多模态模型进入真实网页操作评测。
3. 揭示视觉 grounding 和动作定位是关键瓶颈。

## 对 Agent Harness 的启发

浏览器 Harness 应同时保留截图、可访问性树、DOM、坐标和动作记录，以便多模态模型定位和复盘。

## 关联笔记

- [[webarena]]
- [[mind2web]]
- [[osworld]]

## 局限与待验证

- 多模态输入成本高，长任务中的视觉上下文管理仍困难。
- 视觉任务成功率受截图分辨率、裁剪和元素遮挡影响。

