---
title: "Mind2Web — Towards a Generalist Agent for the Web"
type: paper-summary
tags:
  - paper
  - web-agent
  - benchmark
  - agent
  - ui-automation
created: 2026-06-14
updated: 2026-06-14
source: "assets/papers/2023-mind2web.pdf"
aliases:
  - "Mind2Web"
---

# Mind2Web: Towards a Generalist Agent for the Web

> **arXiv**：https://arxiv.org/abs/2306.06070  
> **PDF 路径**：`assets/papers/2023-mind2web.pdf`

## 核心思想

Mind2Web 提供真实网站上的 Web Agent 数据集，要求 Agent 根据自然语言任务在网页中选择元素并执行操作。它关注跨网站、跨领域的泛化能力。

## 任务形式

- 输入：用户目标、网页状态、候选元素。
- 输出：下一步操作，例如点击、输入、选择。
- 场景：旅游、购物、社交、金融、服务预订等真实网站。

## 贡献

1. 把 Web Agent 评测从小型模拟环境推进到真实网站数据。
2. 强调 HTML/DOM 结构理解和动作选择。
3. 揭示跨网站泛化仍然困难。

## 对 Agent Harness 的启发

浏览器 Agent 需要稳定的页面表示、元素定位、动作回放、失败重试和状态快照。只给截图或只给 DOM 都不够。

## 关联笔记

- [[webarena]]
- [[visualwebarena]]
- tool-use-system

## 局限与待验证

- 数据集偏离线评测，不能完全覆盖真实浏览器动态交互。
- 登录、验证码、个性化页面和实时变化会增加部署难度。

