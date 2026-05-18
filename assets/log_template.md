# Wiki 操作日志（append-only）

> 一行一记录，按时间倒序追加。统一前缀方便 `grep "^## \[" log.md | tail -20` 查看。

---

## [YYYY-MM-DD] init | 初始化 wiki 骨架
- 创建目录 raw/, wiki/, wiki/sources/, wiki/entities/, wiki/concepts/, wiki/analysis/, wiki/_meta/
- 创建 SCHEMA.md（待填充）
- 创建 index.md
- 创建 log.md

## [YYYY-MM-DD] ingest | <原文标题>
- source: [[source-slug]]
- 新建页面 (N): [[xxx]], [[yyy]]
- 更新页面 (M): [[zzz]] (+断言3条), [[aaa]] (修正时间线)
- 发现冲突: [[bbb]] 与新资料对 …… 说法不同 → 已标注
- touched 共 P 页

## [YYYY-MM-DD] query | <问题简述>
- 读取页面: [[xxx]], [[yyy]]
- 答案是否回填: 是 → [[analysis-slug]]

## [YYYY-MM-DD] lint | 健康检查
- 报告: [[lint-YYYY-MM-DD]]
- 发现 12 处问题，已自动修复 5 处，待用户裁决 7 处

## [YYYY-MM-DD] schema | SCHEMA 演进
- 新增 module 实体类型
- 调整 ingest 默认偏好为自动建实体页

---

## 日志条目格式约定

```
## [YYYY-MM-DD] <op> | <一句话主题>
- 简明列出本次动作的输入、输出、注意事项
```

`<op>` 取值：`init / ingest / query / lint / schema / refactor / cleanup`。