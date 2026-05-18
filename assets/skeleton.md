# 初始化骨架

第一次为某个 `<wiki_root>` 构建 wiki 时，按下面创建文件/目录。

## 目录

```
<wiki_root>/
├── raw/                    # 原始资料（不可变）
│   └── assets/             # 图片等附件
├── wiki/
│   ├── sources/            # 每条资料的摘要页
│   ├── entities/           # 实体页
│   ├── concepts/           # 概念页
│   ├── analysis/           # 综合/对比/推论
│   └── _meta/              # lint 报告、回顾
├── SCHEMA.md
├── index.md
└── log.md
```

## 三个根级文件的初始内容

### `SCHEMA.md`（占位，与用户共填）
参见 `references/schema-conventions.md` 模板。

### `index.md`（初始）
参见 `assets/index_template.md`。

### `log.md`（初始）
参见 `assets/log_template.md`，写入第一条：

```
## [YYYY-MM-DD] init | 初始化 wiki 骨架
- 创建目录 raw/, wiki/, wiki/sources/, wiki/entities/, wiki/concepts/, wiki/analysis/, wiki/_meta/
- 创建 SCHEMA.md（待填充）
- 创建 index.md
- 创建 log.md
```

## 给用户的初始化提示

完成骨架创建后向用户说明：

1. 把要进入 wiki 的原始资料放到 `raw/` 下（保持原文件名，便于追溯）。
   - `raw/` **支持任意层级子目录**，可以按类型、时间或项目组织，例如：
     ```
     raw/
     ├── papers/2024/attention-is-all-you-need.pdf
     ├── podcasts/lex-fridman/ep-123.md
     ├── projects/xauto/design.md
     ├── chats/2025-Q1/discussion.md
     └── assets/                  # 全局图片附件
         └── projects/xauto/      # 也可以就近建子目录附件
     ```
   - 选定一种组织方式后，建议在 `SCHEMA.md` 的 "raw 目录划分约定" 一节中记录下来，便于以后保持一致。
2. 共同填好 `SCHEMA.md`（领域、目的、命名约定、raw 目录划分、工作流偏好）。
3. 选择启动方式：
   - **盘点式**：让 agent 扫描已存在的 raw/ 资料（递归遍历所有子目录），建立第一版 wiki。
   - **增量式**：从下一条新资料开始一条条 ingest。
4. 推荐使用 Obsidian 打开 `<wiki_root>`，便于图谱视图浏览。