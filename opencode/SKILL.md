---
name: memory
description: "Use when the user asks to remember, recall, search, update, or forget/delete personal notes, preferences, coding conventions, project context, past decisions, or any explicitly stored information. Triggers: remember, recall, memory, forget, delete memory, note this, what did I say, what did we decide, look up, my notes."
---

# Memory Skill

你正在管理一个个人知识库——用户明确要求你记住的内容，包括编码偏好、项目背景、工具配置、历史决策等。

## 存储位置

所有记忆数据存放在 `~/.agents/memories/data/`，索引文件为 `~/.agents/memories/INDEX.md`。每个 `.md` 文件覆盖一个主题/类别。

## 条目格式

每条记忆遵循以下结构：

```
### [YYYY-MM-DD] 简要标题

正文为自由格式的 Markdown。
- 关键点
- 细节
```

- 日期使用 `YYYY-MM-DD` 格式，放在标题开头。
- 标题应简短、便于扫描。
- 添加新条目时使用当前日期。

## 操作

### 1. 查询

当用户要求回忆、查找或搜索某内容时：

1. **查看索引**：`~/.agents/memories/INDEX.md`，其中列出了所有文件及简要描述。
2. **读取匹配的文件**：从 `~/.agents/memories/data/` 中找到后，清晰呈现信息。
3. **必要时 grep**：如果索引和直接读取都找不到，在 `~/.agents/memories/data/` 中全文搜索。
4. **如实报告**：如果没有匹配内容，直接说明。绝不编造或猜测。

### 2. 新增

当用户明确要求记住、记录或保存某内容时：

1. **选择分类文件。**
   - 优先使用索引中已有的合适文件。
   - 否则，在 `~/.agents/memories/data/` 中新建 `kebab-case.md` 文件。
   - 如果不确定，**询问用户**应放入哪个分类。
2. **检查重复。** 快速扫描目标文件。如果该记忆已存在，告知用户并询问：是否改为更新已有条目？
3. **追加条目**到文件末尾，使用上述格式。新条目放在底部。
4. **更新索引**：编辑 `~/.agents/memories/INDEX.md`，严格按以下格式追加一行：
   `- \`文件名.md\` — 简要描述`
5. **确认**保存了什么、保存在哪里。

### 3. 修改

当用户要求更新、更正或修改某记忆时：

1. **搜索**目标条目（参照查询流程）。
2. **确认**：展示片段并询问"是这个吗？"
3. **编辑**：使用编辑工具精确修改，只改用户要求改的内容。
4. **确认**修改结果。

### 4. 删除

当用户要求忘记或删除某记忆时：

1. **搜索**目标条目。
2. **确认**删除前展示片段并询问。
3. **删除**条目。如果文件变为空：
   - 删除 `.md` 数据文件。
   - 编辑 `~/.agents/memories/INDEX.md`，移除对应索引行（INDEX.md 本身禁止删除）。
4. **确认**删除了什么。

## 规则

- **绝不编造。** 如果记忆中没有相关内容，直接说明。
- **一个主题一个文件。** 如果记忆文件超过约 200 行，建议拆分。
- **删除必须确认。** 未经用户确认绝不删除。
- **INDEX.md 禁止删除。** 只允许编辑其内容，绝不删除 INDEX.md 文件本身。
- **不确定时询问。** 分类模糊、疑似重复、指令不清 → 询问用户。
