---
title: "HarnessX"
tags: [agent-harness, harnessx, framework-factory, evolution]
created: 2026-07-29
updated: 2026-07-29
---

# HarnessX

> 可组合、自适应、可演进的智能 Agent 运行框架工厂。将运行框架（harness）提升为一等公民对象，通过替换代数组装类型化组件，内置 AEGIS 多智能体轨迹演化引擎，打通框架-模型协同优化闭环。

## 基本信息

- **仓库**: /Users/hhl/Documents/projs/HarnessX (本地克隆)
- **定位**: Agent 运行框架的工业化生成平台
- **三大支柱**: Compose（组合）、Adapt（自适应）、Evolve（演进）
- **语言**: Python (uv 管理)
- **组件**: harnessx/（核心 SDK）、gateway/（HTTP 网关）、frontend/（Lab UI）、benchmarks/（5 个基准适配器）、recipe/（演化配方）、extensions/（技能/插件）

## 核心架构

```mermaid
graph TD
    subgraph Compose
        HC[HarnessConfig: 行为管道]
        MC[ModelConfig: 模型绑定]
        HB[HarnessBuilder: | 组合]
        P[9维处理器]
    end
    subgraph Adapt
        AEGIS[AEGIS 演化引擎]
        MA[MetaAgent: 元智能体]
        EV[EvolveValidator: 确定性闸门]
        J[Journal: 跨轮记忆]
    end
    subgraph Evolve
        RL[RL 训练桥]
        TR[Trajectory → SFT/GRPO]
        SG[SGLang Provider]
    end
    HC --> H[Harness]
    MC --> H
    H --> RL[run_loop]
    RL --> TR
    TR --> SG
    AEGIS --> MA
    MA --> EV
    EV --> J
    J --> MA
```

## 九维行为分类

| 维度 | 控制 | 代码位置 |
|---|---|---|
| D1 Model | 多 provider 路由 + 角色分配 | ModelConfig |
| D2 Context | 系统提示 + 历史截断 + 用户包装 | processors/context/ |
| D3 Memory | 提取→存储→检索，5 种策略 | processors/memory/ |
| D4 Tools | 内置工具 + MCP + Skills + 过滤 | processors/tools/ + tools/ |
| D5 Sandbox | Local / Docker / E2B 隔离 | sandbox/ |
| D6 Evaluate | LLM Judge / SelfVerify / PRM | processors/evaluation/ |
| D7 Control | 13 个处理器：循环检测、成本守卫、压缩等 | processors/control/ |
| D8 Observe | HarnessJournal JSONL + OTel + 检查点 | processors/observability/ + tracing/ |
| D9 Train | 轨迹→SFT/RL 记录 + token 标注 | rl/ + core/trajectory.py |

## 子系统概览

| 子系统 | 说明 | 详细笔记 |
|---|---|---|
| 框架组合机制 | 8 钩子 + Processor 管道 + HarnessBuilder `|` 组合 | [[harnessx-framework-composition]] |
| Benchmark 评测 | 5 个基准适配器，通用 Task/EvalResult 接口 | [[harnessx-benchmark-evaluation]] |
| AEGIS 演化引擎 | Digester/Planner/Evolver/Critic + 变体隔离 + 协同演化 | [[harnessx-aegis-evolution]] |
| 沙箱/工具/RL | Local/Docker/E2B 沙箱 + MCP + spawn_subagent + GRPO 训练 | [[harnessx-sandbox-tools-rl]] |

## 快速示例

```python
from harnessx import Harness, BaseTask, HarnessBuilder
from harnessx.core.model_config import ModelConfig
from harnessx.providers.litellm_provider import LiteLLMProvider
from harnessx.bundles import context, coding

# 组合行为管道（不含模型）
harness_config = (HarnessBuilder() | context | coding).build()

# 绑定模型
harness = LiteLLMProvider("claude-sonnet-4-6").agentic(harness_config)

# 运行任务
result = await harness.run(BaseTask(
    description="Write a hello-world Flask app",
    max_steps=20, token_budget=80_000, max_cost_usd=0.50,
))
print(result.final_output, result.exit_reason)
```

## 核心实验结果

- 5 基准 × 3 模型，15 组中 14 组正向提升，平均 +14.5%
- 逆缩放效应：基线越弱的模型收益越大（ALFWorld Qwen3.5-9B +44.0%）
- 协同演化额外 +4.7% 增益
- Terminal Bench 2.0: HarnessX 63.0% (k=1) vs Claude Code 58.0% (k=5)

## 参考

- 源码: /Users/hhl/Documents/projs/HarnessX
- 技术报告: 见对话附件
- [[strands-agents-sdk]] — Strands Agents SDK（对比参考）
