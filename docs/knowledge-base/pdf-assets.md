---
title: PDF 与附件维护规范
tags: [knowledge-base, assets, pdf, git-lfs]
created: 2026-06-12
updated: 2026-06-12
---

# PDF 与附件维护规范

本知识库可以保存 PDF、图片和其他附件，但附件只作为原始材料或引用材料存在。可检索、可维护的知识仍应沉淀为 Markdown 笔记。

## 目录约定

PDF 统一放在 `assets/` 下，不直接散落在 `docs/` 目录中：

```text
assets/
├── papers/          # 论文、技术报告、白皮书
├── references/      # 手册、规范、官方资料、长文导出
└── images/          # 图片、截图、图表
```

按主题继续分目录，例如：

```text
assets/papers/agent-harness/
assets/papers/rag-engineering/
assets/references/ai-coding/
```

## 命名规范

文件名使用小写英文、数字和短横线，避免空格和中文标点：

```text
2024-agent-harness-runtime-patterns.pdf
2023-rag-survey.pdf
openai-tool-use-reference.pdf
```

推荐格式：

```text
<year>-<short-title>.pdf
```

如果同一主题下有多个来源，可以追加来源名：

```text
2024-agent-harness-runtime-patterns-anthropic.pdf
2024-agent-harness-runtime-patterns-openai.pdf
```

## Markdown 引用方式

在 Obsidian 中优先使用普通 Markdown 链接，路径从当前笔记相对定位：

```markdown
[PDF 原文](../../assets/papers/agent-harness/2024-agent-harness-runtime-patterns.pdf)
```

如果需要利用 Obsidian 的附件打开能力，也可以使用 wikilink：

```markdown
[[assets/papers/agent-harness/2024-agent-harness-runtime-patterns.pdf]]
```

同一篇 PDF 如果被多篇笔记引用，不要复制多份文件。保留一个原始 PDF，并在相关 Markdown 笔记中链接它。

## 摘要卡片

重要 PDF 应配套一篇 Markdown 摘要卡片，放在对应知识主题的 `docs/` 目录中。摘要卡片负责进入知识图谱，PDF 负责保存原始材料。

摘要卡片建议包含：

- 原文链接或本地 PDF 链接
- 核心结论
- 关键概念
- 与现有笔记的关联
- 可信度、适用范围和待验证问题

示例：

```markdown
---
title: Agent Harness Runtime Patterns
tags: [agent-harness, runtime, paper]
created: 2026-06-12
updated: 2026-06-12
source: ../../assets/papers/agent-harness/2024-agent-harness-runtime-patterns.pdf
---

# Agent Harness Runtime Patterns

原文：[PDF](../../assets/papers/agent-harness/2024-agent-harness-runtime-patterns.pdf)

## 核心结论

...

## 关联笔记

- [[harness-architecture]]
- [[tool-use-system]]
```

## Git LFS 规则

仓库已通过 `.gitattributes` 配置 PDF 使用 Git LFS：

```gitattributes
*.pdf filter=lfs diff=lfs merge=lfs -text
```

因此提交 PDF 前应确认本机已安装 Git LFS：

```powershell
git lfs version
```

首次使用某个仓库时建议执行：

```powershell
git lfs install
```

提交 PDF 后可以检查文件是否走 LFS：

```powershell
git lfs ls-files
git check-attr -a -- assets/papers/example.pdf
```

## 不建议的做法

- 不把 PDF 直接放在 `docs/` 里。
- 不把同一 PDF 复制到多个主题目录。
- 不只提交 PDF 而没有任何 Markdown 摘要或索引入口。
- 不把临时下载、重复版本、未确认来源的 PDF 长期保留在仓库。
- 不把个人阅读批注工具产生的临时文件提交到仓库。
