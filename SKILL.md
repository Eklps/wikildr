---
name: wikildr
description: 基于 Andrej Karpathy 的 llm-wiki 理念，把零散原始资料（文章、论文、播客笔记、聊天记录、PDF、代码文档等）增量编译成一个持久化、互链、可演进的个人/项目知识库 Wiki。当用户提到"构建知识库 / 知识库 / wiki / llm-wiki / 笔记整理 / 阅读笔记沉淀 / 把这些资料整理成一个体系 / 把文档结构化 / 增量喂资料并建立交叉引用 / 帮我维护一个 Obsidian 风格的 markdown 知识网络"等需求时，务必激活本 skill。本 skill 覆盖三大操作：Ingest（摄取新资料并更新相关页面）、Query（基于 wiki 回答问题并把高价值答案回填）、Lint（健康检查：发现矛盾、过时声明、孤儿页面、缺失交叉引用）。默认使用中文输出。
metadata:
  short-description: 增量构建并持续维护 Karpathy 风格的 LLM Wiki 知识库
---

# wikildr

把原始资料编译成一个**持久化、互链、可演进**的 Markdown 知识库，由你（agent）负责所有"枯燥的"摘要、交叉引用、归档与一致性维护工作，让用户专注于资料筛选与提问。

> 核心理念：Wiki 是一个**逐步累积的产物**，不是查询时临时拼凑的检索结果。原始资料是"源代码"，wiki 是被你持续编译、刷新的"产物"，schema 是约束你行为的"构建脚本"。

---

## 1. 三层架构（必须严格区分）

| 层 | 路径 | 谁来写 | 是否可变 |
|---|---|---|---|
| **原始资料层 Raw** | `<wiki_root>/raw/` | 用户投放，agent 只读 | 不可变（append-only） |
| **Wiki 层** | `<wiki_root>/wiki/` | agent 完全拥有，用户基本不写 | 完全可重写 |
| **Schema 层** | `<wiki_root>/SCHEMA.md`、`<wiki_root>/index.md`、`<wiki_root>/log.md` | agent 维护，用户与你共同演进 SCHEMA.md | 持续演进 |

`<wiki_root>` 默认是当前工作区的根目录。如果用户没有显式说明，**第一次激活时必须先与用户确认根目录**，再在该目录下创建 `raw/`、`wiki/`、`SCHEMA.md`、`index.md`、`log.md` 五个最小骨架（参考 [assets/skeleton.md](assets/skeleton.md)）。

⚠️ 永远不要修改 `raw/` 下的任何文件。如果原文有错，在 wiki 中标注；不动原文是为了保留可溯源性。

---

## 2. 何时触发本 skill

满足以下任一情况就应该把本次任务当成 wiki 操作：

- 用户给出新资料并暗示/明示"把它整理进来"、"沉淀一下"、"加入知识库"
- 用户问的问题在已有 wiki 中可能有答案，或答案值得被回填
- 用户要求"梳理"、"健康检查"、"看看哪里有矛盾 / 过时 / 孤立"
- 用户要求基于已有资料生成新产物（对比表、综述、幻灯片 Marp、图表）

如果用户只是闲聊或一次性问答且明确不希望沉淀，可以不走 wiki 流程。

---

## 3. 三大操作

每次操作都遵循同一闭环：**读 SCHEMA → 读 index.md → 读相关页面 → 执行变更 → 更新 index.md → 追加 log.md**。

### 3.1 Ingest（摄取新资料）

输入：用户在 `raw/` 中投放（或在对话中粘贴）了新资料。

步骤（推荐与用户逐条确认，而不是无声完成）：

1. **读取并理解原文**：阅读 `raw/<source>`，提取实体（人物、组织、产品、概念）、断言（claim）、时间戳、来源链接。
2. **与用户对齐重点**：用 3–5 句话向用户复述关键 takeaway，确认你抓到的重点和他一致。
3. **写源摘要页**：在 `wiki/sources/<slug>.md` 创建对应摘要页，frontmatter 包含 `title / source_path / date / tags / extracted_entities`。模板见 [assets/page_source.md](assets/page_source.md)。
4. **更新相关实体/概念页**：对原文中提到的每个实体/概念，**要么扩展已有页面**、**要么新建页面**。新断言要带 `[[source-slug]]` 反向引用，矛盾要显式标注 `> ⚠️ 与 [[xxx]] 的旧说法冲突`。
5. **维护交叉引用**：用 Obsidian 风格 `[[wiki-link]]` 连接所有相关页面。新概念出现就建页，宁可页面短，不要让它孤立散落在某段正文里。
6. **更新 `index.md`**：在合适分类下加入新页面条目（链接 + 一句话简介）。
7. **追加 `log.md`**：写一条 `## [YYYY-MM-DD] ingest | <source title>`，列出新建/修改了哪些页面（典型一次 ingest 会触达 8–15 个页面，这是正常的）。

一次 ingest 默认**串行一条资料**，便于用户审阅。除非用户明确说"批量"，否则不要一次喂多条。

### 3.2 Query（基于 wiki 回答）

输入：用户提出一个问题。

步骤：

1. 先读 `index.md`，定位候选页面（不要先做向量检索，规模在数百页内时 index + grep 就够）。
2. 必要时再读具体页面与其 `[[link]]` 邻居。
3. 综合作答，**所有事实性陈述都带 `[[wiki-link]]` 或 `(raw/xxx.md)` 引用**，不要凭空生成。
4. 判断这次答案是否值得回填：
   - 是新的对比、综合、推论 → 在 `wiki/analysis/<slug>.md` 建一个新页面，更新 index 与 log。
   - 只是复述已有信息 → 不必回填，避免冗余。
5. 输出形式可以是 markdown、表格、Marp 幻灯片、matplotlib 图等，但**任何沉淀产物都应该是文件，而不是只存在于对话中**。

### 3.3 Lint（健康检查）

定期或在用户要求时执行。检查清单见 [references/lint-checklist.md](references/lint-checklist.md)，要点：

- 跨页面**矛盾**：同一断言在 A 页与 B 页说法不同。
- **过时声明**：被更新的资料覆盖但旧页未刷新。
- **孤儿页**：无入站链接的页面。
- **缺失交叉引用**：A 页提到 X，但没有指向 `[[X]]`。
- **应建未建的页面**：被多次提及的概念却没有自己的页面。
- **数据缺口**：某主题下断言稀疏，建议追加新资料。

输出一份 lint 报告页 `wiki/_meta/lint-<date>.md`，并按用户确认逐项修复。

---

## 4. 索引与日志（基础设施）

### `index.md`（**内容导向**）
按分类组织：`## Sources / ## Entities / ## Concepts / ## Analysis / ## Meta`。每条目格式：
```
- [[page-slug]] — 一句话说明（可选元数据：日期 / 来源数 / 标签）
```
模板见 [assets/index_template.md](assets/index_template.md)。

### `log.md`（**时间导向，append-only**）
统一前缀，便于 `grep "^## \[" log.md | tail -20` 查看最近活动：
```
## [2026-05-18] ingest | XAuto 仿真平台架构文档
## [2026-05-18] query  | 回归测试与 CICD 的关系
## [2026-05-18] lint   | 发现 3 处矛盾、2 个孤儿页
```
模板见 [assets/log_template.md](assets/log_template.md)。

---

## 5. SCHEMA.md：本 wiki 的"构建脚本"

`SCHEMA.md` 记录用户与你共同约定的：

- 本 wiki 的领域与目的（个人研究 / 项目知识库 / 读书伴随 wiki / 团队内部 wiki）
- 实体类型（如：人物 / 组织 / 工具 / 项目 / 模块 / 概念 / 事件）
- 页面 frontmatter 字段约定
- 命名约定（slug 规则、目录划分）
- 摄取/查询/lint 的偏好（是否自动建页、是否每次都让用户确认）

详细约定项与默认值见 [references/schema-conventions.md](references/schema-conventions.md)。**SCHEMA.md 是会演进的**——遇到反复出现的判断分歧，主动建议把它固化进 SCHEMA。

---

## 6. 页面写作风格（很关键）

- **每页有清晰焦点**：一个实体 / 一个概念 / 一个资料 / 一次分析。不要做"大杂烩页"。
- **断言尽量原子化**：一句一断言，便于后续修改和交叉引用。
- **永远带来源**：每个非显然的断言后面跟 `[[source-slug]]` 或 `(raw/xxx.md#L23)`。
- **冲突显式标注**：发现新资料推翻旧说法时，不要静默覆盖，用引用块标注两方说法，让用户裁决或保留两说。
- **小步快跑**：宁可建 10 个简短页面 + 大量互链，也不要一个膨胀的长文档。
- **中文为主**：除非用户另有要求，所有 wiki 内容用中文，但保留专有名词、代码、API 名的原文。

页面模板见 `assets/page_*.md`。

---

## 7. 与 Obsidian / Git 的协作

- wiki 就是一堆 markdown 文件 + 一个 git 仓库。用户可能开着 Obsidian 浏览。
- 使用 `[[wiki-link]]` 而不是 `[text](path.md)`，让图谱视图工作。
- 图片/附件放在 `raw/assets/`，wiki 页面引用时用相对路径。
- 大动作（结构重排、批量改名）前提醒用户先 commit，便于回滚。

---

## 8. 常见反模式（不要做）

- ❌ 在对话里给出"漂亮的总结"但不写进 wiki —— 这违背了"持久化、累积"的核心理念。
- ❌ 一次 ingest 偷偷修改 30+ 文件却不在 log 中记录。
- ❌ 用一个长 `index.md` 代替正确的多页面 + 互链结构。
- ❌ 修改 `raw/` 下的文件。
- ❌ 对话中临时检索后给答案，但不读 wiki，导致 wiki 永远不被使用 → 用户失去信任。
- ❌ 静默覆盖旧断言。新旧冲突必须显式标注。

---

## 9. 启动模板

第一次为某个目录构建 wiki 时，按以下顺序执行：

1. 与用户确认 `<wiki_root>` 和领域目的。
2. 用 [assets/skeleton.md](assets/skeleton.md) 初始化目录骨架。
3. 与用户共同填写 `SCHEMA.md`（基于 [references/schema-conventions.md](references/schema-conventions.md) 模板）。
4. 询问是先做一次 **盘点式 ingest**（扫描已有的 raw/ 资料，建立第一版页面）还是从下一份资料开始增量构建。
5. 第一次 ingest 完成后，主动建议用户检查 Obsidian 图谱视图，确认结构看起来合理。

---

## 10. 何时建议引入额外工具

默认 index + grep 足够支撑数百页规模。当 wiki 增长到以下程度时，向用户建议（不要擅自引入）：

- 页面 > 300：建议引入 [`qmd`](https://github.com/tobi/qmd) 或类似 BM25+向量混合检索工具。
- 出现大量频繁查询的固定问题：建议把它们沉淀成 `wiki/faq/<slug>.md` 页面而不是依赖检索。
- 多人协作：建议把 wiki 仓库化（Git）并讨论 review 流程。

工具是可选的、模块化的，按需引入，不要一开始就堆基础设施。

---

## 参考资料

- [references/schema-conventions.md](references/schema-conventions.md) — SCHEMA.md 的字段约定与默认值
- [references/lint-checklist.md](references/lint-checklist.md) — lint 阶段完整检查项
- [assets/skeleton.md](assets/skeleton.md) — 初始化目录骨架与内容
- [assets/page_source.md](assets/page_source.md) — 源摘要页模板
- [assets/page_entity.md](assets/page_entity.md) — 实体页模板
- [assets/page_concept.md](assets/page_concept.md) — 概念页模板
- [assets/index_template.md](assets/index_template.md) — index.md 模板
- [assets/log_template.md](assets/log_template.md) — log.md 模板