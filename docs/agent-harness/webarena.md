---
title: "WebArena — A Realistic Web Environment for Building Autonomous Agents"
type: paper-summary
tags:
  - paper
  - web-agent
  - benchmark
  - agent
  - environment
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-webarena.pdf"
aliases:
  - "WebArena"
---

# WebArena: A Realistic Web Environment for Building Autonomous Agents

> **arXiv**：https://arxiv.org/abs/2307.13854  
> **PDF 路径**：`assets/papers/2023-webarena.pdf`

## 核心思想

WebArena 构造一组可复现的真实风格网站环境，用于评估自主 Web Agent 的端到端任务完成能力。它比离线元素选择更接近真实网页操作。

## 环境组成

- 电商网站
- 社交论坛
- 协作开发平台
- 内容管理系统
- 地图/信息网站

## 贡献

1. 提供可控、可复现、可执行的 Web Agent benchmark。
2. 任务需要多步操作、信息检索、表单填写和跨页面导航。
3. 证明当前 LLM Agent 在真实网页任务上仍有明显差距。

## 对 Agent Harness 的启发

端到端评测需要真实环境、状态重置、账号隔离、动作日志和任务判定器。Harness 不能只评测单步选择。

## 关联笔记

- [[mind2web]]
- [[visualwebarena]]
- [[agentbench]]

## 局限与待验证

- 环境维护成本高，网站版本和依赖会影响复现。
- 成功判定器设计会直接影响 benchmark 可信度。

