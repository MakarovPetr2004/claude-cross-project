# claude-cross-project

A Claude Code plugin that lets you explore and edit sibling projects using parallel subagents — with Russian and English command support.

## What it does

When your task spans multiple projects, this skill spawns parallel subagents to:
- **Explore** sibling projects (fast, read-only, haiku model)
- **Edit** code in other projects (sonnet model, isolated in worktree)
- **Save** cross-project interaction knowledge to memory

All projects are discovered automatically from the parent directory of your current project.

## Installation

### From GitHub (recommended)
```bash
git clone https://github.com/Nintiko/claude-cross-project.git
cc --plugin-dir /path/to/claude-cross-project
```

### One-liner
```bash
git clone https://github.com/Nintiko/claude-cross-project.git ~/.claude/plugins/cross-project
cc --plugin-dir ~/.claude/plugins/cross-project
```

## Usage

### Exploration
```
/cross-project union
/cross-project union academy_frontend
исследуй union
изучи union и academy_frontend
```

### Editing
```
/cross-project union --edit
отредактируй union
```

### Save to memory
```
/cross-project --remember
запомни взаимодействие
исследуй union --запомни
```

## How it works

The skill uses `pwd` to detect the current project directory and automatically resolves sibling projects one level up. Works on Windows (bash), macOS, and Linux.

| Mode | Subagent model | Tools |
|------|---------------|-------|
| Explore | haiku (fast) | Read, Grep, Glob |
| Edit | sonnet | Read, Grep, Glob, Edit, Write, Bash |

## Requirements

- [Claude Code](https://claude.ai/claude-code) installed
