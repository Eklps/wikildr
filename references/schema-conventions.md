# SCHEMA.md 字段约定与默认值

每个 wiki 实例都必须有一份 `SCHEMA.md`。它是用户与 agent 共同维护的"构建脚本"，定义本 wiki 的形态与维护规则。**SCHEMA 不是一次性的，要在使用中不断演进。**

下面是建议字段与默认值。复制为模板填到 `<wiki_root>/SCHEMA.md`。

---

## 1. 基本信息

```yaml
---
wiki_name: ""              # 例如：xauto-knowledge / 我的研究 / 战争与和平阅读笔记
domain: ""                 # 个人 / 研究 / 项目 / 团队 / 读书 / 其他
purpose: ""                # 一两句话说明：这个 wiki 是为了什么
created_at: "2026-05-18"
maintainer_agent: "JoyCode"
language: "zh-CN"          # 输出默认语言
---
```

## 2. 目录约定

| 目录 | 用途 |
|---|---|
| `raw/` | 原始资料（不可变） |
| `raw/assets/` | 原文中提取的图片/附件 |
| `wiki/sources/` | 每条原始资料对应一个摘要页 |
| `wiki/entities/` | 实体页面（人、组织、产品、项目、模块） |
| `wiki/concepts/` | 概念页面（抽象术语、方法、模式） |
| `wiki/analysis/` | 综合分析、对比、推论（由 query 阶段沉淀） |
| `wiki/faq/` | 高频问题答案沉淀（可选） |
| `wiki/_meta/` | lint 报告、回顾、SCHEMA 变更记录 |

允许根据领域增设目录（例如读书 wiki 可加 `wiki/characters/`、`wiki/themes/`、`wiki/plot/`）。新增目录时在 SCHEMA 中登记。

## 3. 命名约定

- 文件名一律用 **kebab-case slug**：`xauto-cicd-flow.md`，避免空格、中文文件名。
- 中文页面 title 写在 frontmatter `title:` 字段里，slug 用拼音或英译。
- 同名概念在不同上下文下要区分：`agent-llm.md` vs `agent-rl.md`，并在两页互相 `[[link]]`。

## 4. 实体类型枚举

约定本 wiki 关心哪些实体类型；ingest 时遇到非清单上的类型，先与用户确认是否扩展枚举。

默认枚举（视领域裁剪）：

- `person` 人物
- `organization` 组织/团队
- `product` 产品/工具
- `project` 项目
- `module` 模块/子系统
- `concept` 概念/术语
- `event` 事件/里程碑
- `paper` 论文/报告

## 5. 页面 frontmatter 规范

所有 wiki 页面必须有 frontmatter，至少包含：

```yaml
---
title: ""             # 显式标题（用于显示）
type: ""              # source / entity / concept / analysis / faq / meta
slug: ""              # 文件名（不含扩展名）
tags: []
sources: []           # [[source-slug]] 列表
updated_at: "2026-05-18"
---
```

可选字段：`confidence: high/medium/low`、`status: stub/draft/stable`、`aliases: []`。

## 6. 工作流偏好（用户与你共同决定）

下列开关由用户在初始化时表态，agent 之后默认遵循：

| 偏好 | 选项 | 默认 |
|---|---|---|
| 是否自动新建实体页 | 自动 / 询问 | 询问 |
| ingest 是否串行 | 串行 / 批量 | 串行 |
| 矛盾处理 | 标注两方 / 立即裁决 / 立即覆盖 | 标注两方 |
| 是否在每次 ingest 后展示 diff 摘要 | 是 / 否 | 是 |
| query 答案是否默认回填 | 默认回填 / 询问 / 默认不回填 | 询问 |
| 中英文混合策略 | 全中 / 全英 / 中文为主保留英文专名 | 中文为主 |

## 7. SCHEMA 变更记录

任何对 SCHEMA 的修改在文件末尾追加一行：

```
## changelog
- 2026-05-18: 初版
- 2026-05-20: 新增 module 实体类型，扩展 wiki/modules/ 目录
```

---

## 何时主动提议修改 SCHEMA

- 同一类判断你已经向用户问过两次以上 → 建议把决策固化进 SCHEMA。
- 某个目录文件数 > 50 → 建议在 SCHEMA 中加二级分类。
- 出现新类型的实体重复出现 → 建议扩展枚举。