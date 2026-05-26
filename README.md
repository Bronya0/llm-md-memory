# llm-md-memory

Give any LLM agent long-term memory — file-driven, zero dependencies.

A memory system built on plain Markdown files. An index + data folder structure lets agents remember your preferences, project context, API keys, and past decisions across sessions.

[中文文档](README_zh.md)

## How it works

The agent checks `INDEX.md` before acting on anything involving preferences, project history, or recurring decisions. If something looks relevant, it reads the corresponding file under `data/`. That's it.

No database, no embeddings, no API calls. Just the agent's own language understanding and a few markdown files.

```
Agent System Prompt (always loaded)
  │
  ├─ Trigger? (new project / preference question / "same as last time" ...)
  │   └─ Yes → read INDEX.md
  │            │
  │            └─ Match? → read data/xxx.md (2 hops, done)
  │            └─ No match? → grep data/ for keywords
  │
  └─ No → skip, zero cost
```

### Key design decisions

- **Index lives outside the agent's system prompt.** Adding or removing memories only touches `INDEX.md` — the agent's config stays stable.
- **INDEX.md is never deleted.** It can be edited, but must always exist. This guarantees the agent always has a starting point.
- **2 hops, not 3 or 4.** The index lists every memory file with a one-line description. The agent sees everything at a glance.
- **Complex operations go through the skill file.** Creating, updating, and deleting memories follow standardized workflows defined in `SKILL.md`.

### Why not SQLite FTS5?

FTS5 gives you BM25-ranked full-text search. It's fast and accurate. But:

- You now have a binary database file to maintain and migrate.
- The agent needs to know (or be taught) how to query it.
- It's keyword-level matching, not semantic. "Storage preference" won't match "save data" without a synonym table.
- Extra dependency for something the agent's own reading comprehension already handles.

### Why not vector embeddings?

Vector search is the default choice for RAG. It's great at semantic matching. But:

- Every retrieval call costs an embedding API request (or local GPU memory).
- You need a vector store — Chroma, Milvus, Pinecone, whatever — deployed somewhere.
- Embedding quality dictates recall quality. Switch models, rebuild the entire index.
- Retrieval is opaque. You don't know why something was or wasn't recalled.

For agent memory (tens to low hundreds of entries), the gap between full-text and semantic search is small. Sometimes the simplest thing that works is the right thing.

## Installation

Copy the following into any agent that supports system prompt instructions:

> Please install the llm-md-memory system.
> Repo: https://github.com/Bronya0/llm-md-memory
>
> Installation steps (idempotent, skip if already exists):
>
> 1. If `~/.agents/memories/` does not exist:
>    ```
>    git clone https://github.com/Bronya0/llm-md-memory.git ~/.agents/memories
>    ```
>    If it already exists:
>    ```
>    git -C ~/.agents/memories pull
>    ```
>
> 2. Agent config file locations:
>    - opencode:     config at `~/.config/opencode/AGENTS.md`, copy `~/.agents/memories/SKILL.md` to `~/.agents/skills/memory/SKILL.md`
>    - Claude Code:  config at `~/.claude/CLAUDE.md`, copy `~/.agents/memories/SKILL.md` to `~/.claude/skills/memory/SKILL.md`
>    - Codex:        config at `~/.config/codex/AGENTS.md`, copy `~/.agents/memories/SKILL.md` to `~/.codex/skills/memory/SKILL.md`
>
> 3. Check whether the config file (`AGENTS.md` / `CLAUDE.md`) already has the memory check rules. If it does, skip. Otherwise append:
>
>    ```
>    ## Memory Check
>
>    Before doing any of the following, read `~/.agents/memories/INDEX.md` to check for relevant memories:
>
>    - Starting a new project or picking up an existing one
>    - Making a tech choice or architectural decision
>    - The user mentions something previously discussed
>    - Uncertain about preferences (formatting style, naming conventions, tool choices, etc.)
>    - The user says "same as last time" or similar
>
>    Memory data lives in `~/.agents/memories/data/`.
>    When adding, modifying, or deleting memories, load `~/.agents/memories/SKILL.md` as the operation guide.
>    ```
>
> 4. Restart the agent for the changes to take effect.

## Directory structure

```
~/.agents/memories/
├── INDEX.md              ← memory index (always loaded first)
└── data/                 ← memory files
    ├── project-prefs.md
    ├── api-keys.md
    └── ...
```

## Integrating with other agents

### opencode

Edit `~/.config/opencode/AGENTS.md` and add the trigger rules from step 3 above.

### Claude Code

Edit `~/.claude/CLAUDE.md` and add the same trigger rules.

### Cursor

Add the trigger rules to your `.cursorrules` file or Cursor Settings → Rules for AI.

### Custom / other agents

Two lines in your system prompt are enough:

1. When encountering preference, history, or project-related questions → read `~/.agents/memories/INDEX.md` first.
2. When modifying memories → load `~/.agents/memories/SKILL.md` for the operation workflow.

## Maintenance

| Operation | What to do | Files affected |
|-----------|-----------|----------------|
| **Query** | Read INDEX.md → load matching data files | Read-only |
| **Add** | Append entry to data file → update INDEX.md | data/ + INDEX.md |
| **Edit** | Edit entry in data file | data/ |
| **Delete** | Remove entry → update INDEX.md (confirm first) | data/ + INDEX.md |

Detailed workflows are in [SKILL.md](SKILL.md). Key rules:

- INDEX.md must **never** be deleted — edit only.
- Index entries use the format: `- \`filename.md\` — short description`
- Deletion requires explicit user confirmation.

## License

GPL-3.0
