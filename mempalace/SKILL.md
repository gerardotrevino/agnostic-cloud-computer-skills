---
name: mempalace
description: How to install, configure, and use MemPalace for persistent semantic memory on a cloud computer. Covers installation, identity setup, mining, searching, and the two-layer context strategy.
author: Gerardo Treviño
created: 2026-06-27
---

# MemPalace — Persistent Semantic Memory

MemPalace gives agents a persistent, searchable memory that survives across sessions. It is a local ChromaDB vector store that indexes your project files and docs. No API key required.

## Installation

```bash
# Requires uv (Python package manager)
uv tool install mempalace
export PATH="$HOME/.local/bin:$PATH"

# Persist PATH in .bashrc
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

## Identity Setup

```bash
mkdir -p ~/.mempalace
cat > ~/.mempalace/identity.txt << 'IDENTITY'
Name: [Your Name]
Organization: [Your Organization]
Role: [Your Role]
Cloud Computer: [Device ID or hostname]
IDENTITY
```

## Initialize a Project Wing

```bash
mempalace init ~/projects/<project-name> --yes --no-llm --auto-mine
```

## Core Commands

| Command | Purpose |
|---------|---------|
| `mempalace wake-up` | Load L0+L1 context at session start (~600–900 tokens) |
| `mempalace mine ~/projects/<name>` | Index project files into the palace |
| `mempalace mine ~/skills/` | Index skills into the palace |
| `mempalace search "query"` | Semantic search across all indexed content |
| `mempalace status` | Show wings, rooms, and drawer counts |

## The Two-Layer Context Strategy

**Layer 1 — Git (durable, shared):** All project context lives in `docs/` committed to Git. This is the source of truth. Any developer on any machine gets full context on `git clone`.

**Layer 2 — MemPalace (local, fast):** A semantic index over Layer 1. Always rebuildable from Git. If the machine is wiped, clone the repo and run `mempalace mine` to restore context in minutes.

The palace is **never committed to Git** — it is local and machine-specific. Context sync between developers happens through Git, not through the palace directly.

## End-of-Session Mining

```bash
# After pushing docs/ changes to Git:
mempalace mine ~/projects/<project-name>
```

Mine after every session where files changed. This keeps the palace current.
