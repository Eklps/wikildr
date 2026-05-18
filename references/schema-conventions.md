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

### 2.1 raw 目录划分约定

`raw/` **支持任意层级的子目录**。用户自行选择组织方式，但应在此节登记下来，避免后续 ingest 时混乱。

常见组织维度（任选一种或组合）：

- **按类型**：`raw/papers/`、`raw/podcasts/`、`raw/chats/`、`raw/docs/`
- **按时间**：`raw/2024/`、`raw/2025-Q1/`
- **按项目**：`raw/projects/xauto/`、`raw/projects/wikildr/`
- **按来源**：`raw/arxiv/`、`raw/twitter/`、`raw/internal/`

填写示例：

```yaml
raw_layout:
  scheme: "type-then-time"        # 自定义说明
  top_level:
    - papers/<year>/              # 论文按年份
    - podcasts/<show-name>/       # 播客按节目
    - projects/<project-name>/    # 项目相关文档
    - chats/<year-quarter>/       # 聊天/讨论记录
  assets_policy: "global"         # global = 统一放 raw/assets/；local = 各子目录就近放 assets/
  slug_prefix_when_collision: true  # 不同子目录下重名文件时，sources slug 加路径前缀
```

> ⚠️ `source_path` frontmatter 和正文 `(raw/...)` 引用**必须使用相对 `<wiki_root>` 的完整路径**，例如 `raw/papers/2024/attention-is-all-you-need.pdf`，不允许省略中间子目录。

## 3. 命名约定

- 文件名一律用 **kebab-case slug**：`xauto-cicd-flow.md`，避免空格、中文文件名。
- 中文页面 title 写在 frontmatter `title:` 字段��，slug 用拼音或英译。
- 同名概念在不同上下文下要区分：`agent-llm.md` vs `agent-rl.md`，并在两页互相 `[[link]]`。
- `wiki/sources/<slug>.md` 的 slug **不要求反映 raw 子目录层级**；但若 `raw_layout.slug_prefix_when_collision: true` 且不同子目录下出现同名文件，应使用带前缀的 slug，例如 `papers-2024-attention-is-all-you-need`、`podcasts-lex-fridman-ep-123`。

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