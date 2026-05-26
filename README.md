# llm-md-memory

给任何 LLM agent 装上记忆能力——纯文件驱动，零依赖。

基于 Markdown 文件的 LLM 记忆系统。通过索引 + 数据文件的简单结构，让 agent 跨会话记住你的偏好、项目背景、API 配置等。

## 安装

将以下内容复制给 AI，即可自动完成安装：

```markdown
请帮我安装 llm-md-memory 记忆系统。
仓库地址：https://github.com/Bronya0/llm-md-memory

安装步骤（确保幂等，已存在则跳过）：

1. 如果 ~/.agents/memories/ 不存在，执行：
   git clone https://github.com/Bronya0/llm-md-memory.git ~/.agents/memories
   如果已存在，执行 git -C ~/.agents/memories pull

2. 各 agent 配置文件位置：
   - opencode:     ~/.config/opencode/AGENTS.md，skill 复制到 ~/.agents/skills/memory/SKILL.md
   - Claude Code:  ~/.claude/CLAUDE.md，skill 复制到 ~/.claude/skills/memory/SKILL.md
   - Codex:        ~/.config/codex/AGENTS.md，skill 复制到 ~/.codex/skills/memory/SKILL.md

3. 在上述配置文件中追加以下触发规则（如已存在则跳过）：

       ## 记忆检查

       在做以下事情之前，先读 ~/.agents/memories/INDEX.md 检查是否有相关记忆：

       - 开始一个新项目或接手一个已有项目
       - 做技术选型 / 架构决策
       - 用户提到了之前聊过的内容
       - 遇到不确定的偏好问题（格式化风格、命名规范、工具选择等）
       - 用户说"和上次一样"或类似表达

       记忆数据存放在 ~/.agents/memories/data/。
       新增/修改/删除记忆时，加载 ~/.agents/memories/SKILL.md 作为操作指令。

4. 安装完成后告知用户
```

## 设计原理

**2 跳查询，确保可靠命中：**

```
Agent System Prompt (始终加载)
  │
  ├─ 触发条件匹配？（新项目 / 偏好问题 / "和上次一样" ...）
  │   └─ 是 → 读 INDEX.md
  │           │
  │           └─ 匹配？→ 直接读 data/xxx.md（2 跳即达）
  │           └─ 无匹配？→ grep 全文搜索 data/
  │
  └─ 否 → 略过，零开销
```

**关键设计决策：**

- **索引独立于 agent 规则文件**：INDEX.md 单独存储，增删记忆时只改它，agent 的 system prompt 保持稳定不变
- **INDEX.md 禁止删除**：只允许编辑内容，确保索引文件永远存在
- **2 跳而非 3~4 跳**：INDEX.md 列出所有记忆文件的描述，agent 一眼看清有哪些记忆
- **增删改走 skill**：复杂操作由 SKILL.md 提供标准化流程

## 目录结构

```
~/.agents/memories/
├── INDEX.md              ← 记忆索引
└── data/                 ← 记忆数据
    └── xxx.md            ← 按主题分类的记忆文件
```

## 集成到各 Agent

### opencode

编辑 `~/.config/opencode/AGENTS.md`，插入以下触发规则：

```markdown
## 记忆检查

在做以下事情之前，先读 `~/.agents/memories/INDEX.md` 检查是否有相关记忆：

- 开始一个新项目或接手一个已有项目
- 做技术选型 / 架构决策
- 用户提到了之前聊过的内容
- 遇到不确定的偏好问题（格式化风格、命名规范、工具选择等）
- 用户说"和上次一样"或类似表达

记忆数据存放在 `~/.agents/memories/data/`。
新增/修改/删除记忆时，加载 `~/.agents/memories/SKILL.md` 作为操作指令。
```

### Claude Code

编辑 `~/.claude/CLAUDE.md`，插入同上内容。

### Cursor

在项目根目录的 `.cursorrules` 或 Cursor Settings → Rules for AI 中插入同上内容。

### 通用 Agent / 自定义

在 system prompt 中加入两句核心规则：

1. 遇到偏好、历史、项目相关问题 → 先读 `~/.agents/memories/INDEX.md`
2. 需要修改记忆 → 加载 `~/.agents/memories/SKILL.md` 中的操作流程

## 维护操作

详细流程见 [SKILL.md](./SKILL.md)。简要概览：

| 操作 | 做什么 | 改哪些文件 |
|------|--------|-----------|
| **查询** | 读 INDEX.md → 匹配则读 data/ 文件 | 只读 |
| **新增** | 追加条目到 data/xxx.md → 更新 INDEX.md | data/ + INDEX.md |
| **修改** | 编辑 data/xxx.md 中的条目 | data/ |
| **删除** | 移除条目 → 更新 INDEX.md | data/ + INDEX.md |

**重要规则：**

- INDEX.md 禁止删除，只允许编辑
- 索引条目格式：`- \`文件名.md\` — 简要描述`
- 删除操作必须经用户确认

## License

GPL-3.0
