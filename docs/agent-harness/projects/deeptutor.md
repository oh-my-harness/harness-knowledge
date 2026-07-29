---
title: "DeepTutor — Agentic Personalized Tutoring"
type: paper-summary
tags:
  - paper
  - ai-education
  - agentic-tutoring
  - personalization
source: "assets/papers/deeptutor-agentic-personalized-tutoring.pdf"
aliases:
  - "DeepTutor"
  - "Agentic Personalized Tutoring Framework"
---

# DeepTutor: Towards Agentic Personalized Tutoring

> **作者**：Bingxi Zhao, Jiahao Zhang, Xubin Ren, Zirui Guo, Tianzhe Chu, Yi Ma, Chao Huang (The University of Hong Kong)  
> **机构**：HKU DS  
> **PDF 路径**：`assets/papers/deeptutor-agentic-personalized-tutoring.pdf`

## 核心思想

DeepTutor 是一个**全开源智能体化辅导框架**，通过统一的任务辅导（citation-grounded problem tutoring）与难度校准的题目生成（difficulty-calibrated question generation）形成闭环，实现真正的个性化教育。

## 三大关键技术组件

### 1. 混合个性化引擎 (Hybrid Personalization Engine)
- **静态知识基础 (SKG)**：将教材分解为原子内容单元，构建知识图谱 + 稠密向量的双索引检索
- **动态个人记忆 (DPM)**：使用 **Trace Forest（轨迹森林）** 分层记忆交互历史（三级粒度：会话摘要→中间计划→细粒度执行记录）
- **学习者画像 (Learner Profile)**：包含会话历史 (Dₛ)、薄弱点清单 (Dₓ)、教学自省 (Dᵣ)

### 2. 个性化任务辅导 (Personalized Problem Tutoring)
三阶段流水线：**调查 (Investigate) → 引导求解 (Guided Solving) → 迭代写作 (Iterative Writing)**  
- 每一步都融入学习者上下文，在最近发展区 (ZPD) 内自适应调整讲解深度
- 支持**自适应重规划**和层级压缩技术

### 3. 个性化题目生成 (Personalized Question Generation)
两阶段架构：**想法生成 (Idea Generation) → 批评引导的问答解释生成 (Critic-Guided QAE Generation)**  
- 独立验证器检查事实正确性与教学合理性，防止自我验证偏差

## TutorBench 基准

| 特性 | 数值 |
|---|---|
| 覆盖学科数 | 30 个知识库，5 大学科 |
| 学习者画像 | 90 个（每个 KB 3 个水平） |
| 交互任务 | 270 个 |
| 评估协议 | LLM 驱动的第一人称学生模拟器 |

## 实验结果

| 指标 | 最佳基线 | DeepTutor | 提升 |
|---|---|---|---|
| 总体质量 (OQ) | 3.57 | **3.91** | **+10.76%** |
| 一般推理 (5个基准平均) | — | 提升范围 | **25.69%–32.03%** |

- 在 SF、PER、APP、VID、LD 五个辅导指标上全面领先
- 消融实验证明 SKG（锚定内容）和 DPM（适配学习者）是互补机制

## 意义与贡献

1. 首个统一的闭环智能体辅导框架
2. 提出 Trace Forest 作为细粒度学习者记忆结构
3. 构建 TutorBench 作为以学生为中心的交互式评估基准
4. 个性化方法与通用推理能力双提升

## 关联笔记

- [[docs/agent-harness/_index|Agent Harness]]
- [[docs/rag-engineering/_index|RAG & Knowledge Engineering]]
- [[docs/ai-coding/_index|AI Coding Best Practices]]
- [[README|LLM Wiki 首页]]

## 延伸方向

- 交互式书引擎 (Book Engine) 的纵向评估
- 多通道主动辅导 (TutorBot) 的实际学习效果研究
