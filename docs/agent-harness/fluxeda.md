---
title: "FluxEDA — Unified Stateful Infrastructure for Agentic EDA"
type: paper-summary
tags:
  - paper
  - eda
  - agentic-infrastructure
  - chip-design
  - mcp
source: "assets/papers/fluxeda.pdf"
aliases:
  - "FluxEDA"
  - "Agentic EDA Infrastructure"
---

# FluxEDA: A Unified Execution Infrastructure for Stateful Agentic EDA

> **作者**：Zhengrui Chen, Zixuan Song, Yu Li, Qi Sun, Cheng Zhuo (Zhejiang University)  
> **PDF 路径**：`assets/papers/fluxeda.pdf`

## 核心思想

FluxEDA 为**智能体化电子设计自动化**（Agentic EDA）提供了一个统一的、有状态的（stateful）基础设施底座。通过网关架构 + 持久化后端实例 + 领域技能（Skills）三层设计，使 LLM 智能体能够在生产级商业 EDA 工具上执行多步、跨工具的迭代优化。

## 系统架构（五层）

```
Access Layer         →  MCP Server / Python SDK / C++ SDK
Communication Layer  →  Socket-RPC 结构化请求与响应
Gateway Layer        →  能力注册、协议验证、参数检查、授权
Tool Adaptation Layer→  统一 api_* 方法封装异构 EDA 工具
Runtime Management   →  实例路由、心跳维护、超时控制、异步长任务
```

### 关键特性

1. **有状态执行模型**：后端实例持久化运行，保留设计加载、时序状态、中间结果
2. **渐进式能力暴露**：先提供轻量探测接口（api_ping, api_list_method），按需查询详细调用规范
3. **领域技能 (Skills)**：编码显式工作流知识（任务分解、输入约束、执行顺序、会话感知），与 MCP 层职责分离
4. **生命周期管理**：进程启动 → 实例绑定 → 端口分配 → 就绪检查 → 存活监控 → 清理回收

## 案例分析

### 案例1：自动后布线时序 ECO

| 迭代 | 动作 | Setup WNS | Setup TNS | Hold WHS |
|---|---|---|---|---|
| Baseline | — | -0.81 ns | -37.37 ns | -0.39 ns |
| Iter 1 | 默认 sizing | -0.76 ns | -34.78 ns | -0.38 ns |
| Iter 2 | 激进 sizing | -0.76 ns | -34.84 ns | -0.39 ns (回退!) |
| Iter 3 | 从 Iter1 回滚+修复 hold | -0.76 ns | -34.78 ns | **+0.01 ns** ✅ |

支持分支探索和状态回滚。

### 案例2：Pareto 驱动标准单元子库选择

| 状态 | 面积 (μm²) | WNS | 使用单元数 | 缩减率 |
|---|---|---|---|---|
| Baseline | 14,651.55 | -0.246 ns | 149 | 0% |
| T-Run11 (最终) | **13,659.95** | -0.448 ns | **36** | **75.8%** |

88.4% 恢复时序差距，同时大幅减少单元库规模。

## 意义与贡献

1. 首个面向 Agentic EDA 的有状态统一基础设施
2. 网关架构解耦了工具异构性与智能体编排
3. Skills 层编码领域知识，降低 LLM 隐式推理的不可靠性
4. 实际商业 EDA 环境验证，支持多步分析—执行—优化循环

## 关联笔记

- [[docs/agent-harness/_index|Agent Harness]]
- [[docs/rag-engineering/_index|RAG & Knowledge Engineering]]
- [[README|LLM Wiki 首页]]

## 延伸方向

- 多 EDA 工具跨流程协同优化
- 更丰富的 Skills 库覆盖完整数字芯片设计流程
