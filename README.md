# wikildr

> **wiki + builder** — Incrementally compile your personal knowledge base

An AI agent skill that incrementally builds and maintains a persistent, interlinked, evolving Markdown wiki from your raw sources — based on [Andrej Karpathy's llm-wiki concept](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

## Why Not RAG?

Traditional RAG retrieves and reassembles fragments from scratch on every query. No accumulation, no cross-references, no contradiction tracking — you start from zero every time.

wikildr is different: the LLM **incrementally builds and maintains a persistent wiki**. Each new source doesn't just get indexed — it gets compiled into the existing wiki: updating entity pages, flagging contradictions, filling in cross-references. Knowledge is compiled once and kept current.

```
RAG:     Every query → retrieve fragments → assemble from scratch (no accumulation)
wikildr: Every ingest → compile into wiki → query against a ready-made knowledge network
```

## Three-Layer Architecture

```
<wiki_root>/
├── raw/                    # Raw sources (immutable, agent read-only)
│   ├── papers/2024/        # Arbitrary subdirectories allowed
│   ├── podcasts/<show>/    # Organize by type / time / project as you like
│   ├── projects/<name>/    # Register the layout in SCHEMA.md
│   └── assets/             # Images and attachments (global)
├── wiki/
│   ├── sources/            # Source summary pages
│   ├── entities/           # Entity pages (people, orgs, projects…)
│   ├── concepts/           # Concept pages (terms, methods, patterns…)
│   ├── analysis/           # Syntheses, comparisons, inferences
│   ├── faq/                # High-frequency Q&A deposits
│   └── _meta/              # Lint reports, retrospectives
├── SCHEMA.md               # The "build script" — structure conventions & workflow prefs
├── index.md                # Content-oriented catalog
└── log.md                  # Chronological operation log (append-only)
```

| Layer | Who writes | Mutability |
|---|---|---|
| **Raw** | User deposits; agent read-only | Immutable (append-only) |
| **Wiki** | Agent owns entirely | Fully rewritable |
| **Schema** | Agent maintains; co-evolved with user | Continuously evolving |

## Three Operations

### Ingest (Add New Sources)

Read source → Extract entities & claims → Write summary page → Update related entity/concept pages → Flag contradictions → Update index → Append to log. A single ingest typically touches 8–15 wiki pages.

### Query (Answer from the Wiki)

Read index to locate candidates → Read specific pages → Synthesize answer with `[[wiki-link]]` citations → Backfill high-value answers as new pages.

### Lint (Health Check)

Detect cross-page contradictions, stale claims, orphan pages, missing cross-references, concepts that deserve their own page, data gaps. Output a structured report and fix issues upon confirmation.

## Quick Start

### 1. Install

Copy the `wikildr` directory into your AI agent's skills folder:

```bash
# Claude Code
cp -r wikildr/ .claude/skills/
```

### 2. Initialize Your Wiki

Tell your agent you want to build a knowledge base. It will guide you through:

1. Confirm the wiki root directory
2. Create the directory skeleton
3. Co-fill `SCHEMA.md` (domain, entity types, workflow preferences)
4. Choose a starting mode: **Inventory** (scan existing sources to build the first version) or **Incremental** (start ingesting from the next source)

### 3. Use It

```
You: Add this paper to the wiki / Archive this into the wiki
→ [Ingest] Agent creates summary page, updates entity/concept pages, flags contradictions

You: What's the difference between Transformers and RNNs for long sequences?
→ [Query] Agent synthesizes from wiki pages, backfills a comparison page

You: Run a health check
→ [Lint] Agent outputs a report on contradictions / orphans / gaps
```

## Design Principles

- **Compile, don't retrieve** — Knowledge is compiled once and kept current, not re-derived on every query
- **Flag contradictions explicitly** — Never silently overwrite; mark both sides with `> ⚠️` and let the user decide
- **Small pages, rich links** — Prefer 10 short interlinked pages over one bloated document
- **Atomic claims** — One claim per sentence, each with a source citation
- **Serial ingestion** — Ingest one source at a time by default, so users can review each step
- **Add tools on demand** — index + grep works for hundreds of pages; consider qmd only at scale

## Obsidian-Friendly

- `[[wiki-link]]` bidirectional links work out of the box with graph view
- Complete frontmatter spec, compatible with Dataview plugin
- Images and attachments stored in `raw/assets/`
- The entire wiki is a git repo — version control for free

## Use Cases

| Scenario | Description |
|---|---|
| **Personal research** | Go deep on a topic over weeks — read papers, incrementally build a comprehensive wiki |
| **Reading a book** | File each chapter, auto-build pages for characters, themes, plot threads |
| **Project knowledge base** | Compile scattered docs, designs, and API references into a structured wiki |
| **Team wiki** | LLM handles the cross-referencing and consistency maintenance nobody wants to do |
| **Competitive analysis** | Rapidly accumulate structured comparisons with automatic contradiction flagging |

## File Structure

```
wikildr/
├── SKILL.md                        # Main skill file — complete operational guide
├── assets/
│   ├── skeleton.md                 # Directory skeleton & initial content
│   ├── index_template.md           # index.md template
│   ├── log_template.md             # log.md template
│   ├── page_source.md              # Source summary page template
│   ├── page_entity.md              # Entity page template
│   └── page_concept.md             # Concept page template
└── references/
    ├── schema-conventions.md       # SCHEMA.md field conventions & defaults
    └── lint-checklist.md           # Lint check checklist
```

## Acknowledgments

Core idea from Andrej Karpathy's [llm-wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — shifting the LLM from a "retrieve-from-scratch chatbot" to a "persistent wiki maintainer."

## License

MIT

---

# wikildr

> **wiki + builder** — 增量编译你的个人知识库

基于 [Andrej Karpathy 的 llm-wiki 理念](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)，把零散原始资料增量编译成一个持久化、互链、可演进的 Markdown 知识库。

## 为什么不是 RAG？

传统 RAG 在每次查询时重新检索、重新拼凑。没有积累，没有交叉引用，没有矛盾标注——每次都从零开始。

wikildr 不同：LLM **增量构建并持续维护一个持久化的 wiki**。每加入一份新资料，不仅生成摘要，还会更新相关实体页、标注矛盾、补充交叉引用。知识编译一次，持续保持更新。

```
RAG:   每次查询 → 检索碎片 → 临时拼答案（无积累）
wikildr: 每次摄取 → 编译进 wiki → 查询时直接用现成的知识网络
```

## 三层架构

```
<wiki_root>/
├── raw/                    # 原始资料（不可变，agent 只读）
│   ├── papers/2024/        # 支持任意层级子目录
│   ├── podcasts/<节目>/    # 按类型 / 时间 / 项目自由组织
│   ├── projects/<名称>/    # 在 SCHEMA.md 中登记你的划分方式
│   └── assets/             # 图片等附件（全局）
├── wiki/
│   ├── sources/            # 每条资料的摘要页
│   ├── entities/           # 实体页（人物、组织、项目……）
│   ├── concepts/           # 概念页（术语、方法、模式……）
│   ├── analysis/           # 综合分析、对比、推论
│   ├── faq/                # 高频问题沉淀
│   └── _meta/              # lint 报告、回顾
├── SCHEMA.md               # wiki 的"构建脚本"——结构约定与工作流偏好
├── index.md                # 内容导向的目录页
└── log.md                  # 时间导向的操作日志（append-only）
```

| 层 | 谁来写 | 是否可变 |
|---|---|---|
| **Raw** | 用户投放，agent 只读 | 不可变（append-only） |
| **Wiki** | agent 完全拥有 | 完全可重写 |
| **Schema** | agent 维护，与用户共同演进 | 持续演进 |

## 三大操作

### Ingest（摄取新资料）

读源 → 提取实体与断言 → 写摘要页 → 更新相关实体/概念页 → 标注矛盾 → 更新 index → 追加 log。一次 ingest 典型触达 8–15 个页面。

### Query（基于 wiki 回答）

先读 index 定位 → 读具体页面 → 综合作答（所有事实带 `[[wiki-link]]` 引用）→ 高价值答案回填为新页面。

### Lint（健康检查）

检查跨页面矛盾、过时声明、孤儿页面、缺失交叉引用、应建未建的概念、数据缺口。输出结构化报告，按确认逐项修复。

## 快速开始

### 1. 安装

将 `wikildr` 目录复制到你的 AI Agent 的 skills 目录下：

```bash
# Claude Code
cp -r wikildr/ .claude/skills/
```

### 2. 初始化 wiki

在对话中告诉 agent 你想构建知识库，它会引导你：

1. 确认 wiki 根目录
2. 创建目录骨架
3. 与你共同填写 `SCHEMA.md`（领域、实体类型、工作流偏好）
4. 选择启动方式：**盘点式**（扫描已有资料建立第一版）或**增量式**（从下一条资料开始逐条 ingest）

### 3. 开始使用

```
你：把这篇论文加入 wiki / 把这个文件归档到知识库
→ [Ingest] agent 生成摘要页、更新实体/概念页、标注矛盾

你：Transformer 和 RNN 在长序列建模上有什么区别？
→ [Query] agent 综合已有页面作答，对比分析回填为新页面

你：帮我做一次健康检查
→ [Lint] agent 输出矛盾/孤儿/缺口报告
```

## 设计原则

- **编译而非检索**：知识编译一次、持续保持，不在查询时从零推导
- **冲突显式标注**：新旧矛盾不静默覆盖，用 `> ⚠️` 标注两方说法，由用户裁决
- **小步快跑**：宁可 10 个简短页面 + 大量互链，也不要一个膨胀长文档
- **断言原子化**：一句一断言，带来源引用，便于后续修改和交叉引用
- **串行摄取**：默认一条一条 ingest，便于用户审阅每步变更
- **按需引入工具**：数百页内 index + grep 足够，规模增长后再考虑 qmd 等搜索引擎

## Obsidian 友好

- 使用 `[[wiki-link]]` 双向链接，图谱视图直接可用
- frontmatter 规范完整，兼容 Dataview 插件
- 图片附件统一存放 `raw/assets/`
- 整个 wiki 就是一个 git 仓库，天然支持版本管理

## 适用场景

| 场景 | 说明 |
|---|---|
| **个人研究** | 深入某领域，逐篇读论文，增量构建知识图谱 |
| **读书笔记** | 按章录入，自动建角色/主题/情节页，读完即得完整 companion wiki |
| **项目知识库** | 把散落的文档/设计稿/API 文档编译成结构化 wiki |
| **团队内部 wiki** | LLM 负责维护没人愿意做的交叉引用和一致性 |
| **竞品分析 / 尽职调查** | 快速积累结构化对比，矛盾自动标注 |

## 文件结构

```
wikildr/
├── SKILL.md                        # Skill 主文件——完整的操作指南
├── assets/
│   ├── skeleton.md                 # 初始化目录骨架与内容
│   ├── index_template.md           # index.md 模板
│   ├── log_template.md             # log.md 模板
│   ├── page_source.md              # 源摘要页模板
│   ├── page_entity.md              # 实体页模板
│   └── page_concept.md             # 概念页模板
└── references/
    ├── schema-conventions.md       # SCHEMA.md 字段约定与默认值
    └── lint-checklist.md           # Lint 检查清单
```

## 致谢

核心理念源自 Andrej Karpathy 的 [llm-wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)——将 LLM 从"每次重新检索的聊天机器人"变为"持续维护知识库的 wiki 编辑者"。

## License

MIT