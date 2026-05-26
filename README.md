# llm-md-memory

给任何 LLM agent 装上记忆能力——纯文件驱动，零依赖。

## 是什么

一个基于 Markdown 文件的 LLM 记忆系统。通过索引文件 + 数据文件的简单结构，让 agent 在跨会话之间记住你的偏好、项目背景、API 配置等信息。

**设计目标**：任何 LLM agent（opencode、Claude Code、Cursor、自定义 agent）都能集成，只需在 system prompt 中加几行规则。

## 快速开始

```bash
# 1. 克隆仓库到用户目录
git clone https://github.com/Bronya0/llm-md-memory.git ~/.agents/memories

# 2. 检查目录结构
ls ~/.agents/memories/
# README.md  INDEX.md  memory-skill.md  data/
```

**Windows (PowerShell)**：

```powershell
git clone https://github.com/Bronya0/llm-md-memory.git $env:USERPROFILE\.agents\memories
```

## 目录结构

```
~/.agents/memories/
├── INDEX.md              ← 记忆索引（agent 每次检查是否触发条件后先读它）
├── memory-skill.md       ← 操作指南（query / add / update / delete）
└── data/                 ← 实际记忆数据
    ├── .gitkeep
    └── your-memory.md    ← 按主题分类的 .md 文件（你后续自行创建）
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
- **2 跳而非 3~4 跳**：INDEX.md 直接列出所有记忆文件的描述，agent 一眼看清有哪些记忆
- **增删改走 skill**：复杂操作由 memory-skill.md 提供标准化流程

## 集成到各 Agent

### opencode

编辑 `~/.config/opencode/AGENTS.md`，插入以下内容：

```markdown
## 记忆检查

在做以下事情之前，先读 `~/.agents/memories/INDEX.md` 检查是否有相关记忆：

- 开始一个新项目或接手一个已有项目
- 做技术选型 / 架构决策
- 用户提到了之前聊过的内容
- 遇到不确定的偏好问题（格式化风格、命名规范、工具选择等）
- 用户说"和上次一样"或类似表达

记忆数据存放在 `~/.agents/memories/data/`。如需新增/修改/删除记忆，
将 `~/.agents/memories/memory-skill.md` 的内容作为操作指令加载即可。
```

### Claude Code

编辑 `~/.claude/CLAUDE.md`，插入相同内容（将路径中的 `~` 替换为实际绝对路径，或确认 Claude Code 支持 `~` 展开）。

### Cursor

在项目根目录或 `~/.cursorrules` 中插入上述记忆检查规则。或在 Cursor Settings → Rules for AI 中添加。

### 通用 Agent / 自定义

在任何 system prompt 中加入以下两句核心规则：

1. **触发时查索引**：遇到偏好、历史、项目相关问题时，先读 `~/.agents/memories/INDEX.md`
2. **增删改走 skill**：需要修改记忆时，加载 `~/.agents/memories/memory-skill.md` 中的操作流程

## 维护操作

详细操作流程见 [memory-skill.md](./memory-skill.md)。简要概览：

| 操作 | 做什么 | 改哪些文件 |
|------|--------|-----------|
| **查询** | 读 INDEX.md → 匹配则读 data/ 文件 | 只读，不改 |
| **新增** | 追加条目到 data/xxx.md → 更新 INDEX.md | data/ + INDEX.md |
| **修改** | 编辑 data/xxx.md 中的条目 | data/ |
| **删除** | 从 data/xxx.md 移除条目 → 更新 INDEX.md | data/ + INDEX.md |

**重要规则：**

- INDEX.md 禁止删除，只允许编辑
- 索引条目格式：`- \`文件名.md\` — 简要描述`
- 一个主题一个文件，文件超过 200 行建议拆分
- 删除操作必须经用户确认

## License

MIT
