# Claude Code

## Introduction

**Claude Code** is a CLI tool by **Anthropic** that lets you interact with **Claude** directly from your terminal. It runs in the context of your current project, giving Claude access to your files, git history, and shell.

---

## Slash Commands

Slash commands control Claude Code's behavior from the prompt.

| Command | Description |
|---|---|
| `/agents` | View and configure available agents |
| `/plan` | Enter plan mode before making changes |
| `/usage` | Check token consumption for the session |
| `! <command>` | Run a shell command directly in the session |

---

## Plan Mode

**Plan mode** is a workflow that separates thinking from doing. When activated with `/plan`, Claude explores the codebase and designs an approach **before** writing any code or modifying files.

The workflow has three phases:

- **Phase 1 — Explore**: Read files, search the codebase, understand the context
- **Phase 2 — Design**: Draft an implementation plan and ask clarifying questions
- **Phase 3 — Execute**: Apply changes only after the plan is approved

> ### 💡 **Why use plan mode?**
>
> It prevents Claude from making unwanted changes by locking edits until you explicitly approve the approach.

---

## Custom Agents

Claude Code supports **custom agents** — specialized assistants focused on a specific domain (git, testing, code review, etc.).

- Agents are defined as **Markdown files** in `~/.claude/agents/`
- Each agent has a **persona**, a set of **tools**, and a defined **scope**
- They can be versioned and shared in a dedicated repo (e.g. `agents/claude/`)

### Example agents

- `git-manager` — handles all git operations: commits, branches, push/pull, conflict resolution

---

### 🔑 **Important Keywords:**
`claude-code`, `CLI`, `slash commands`, `plan mode`, `agents`, `Anthropic`, `MCP`, `terminal`
