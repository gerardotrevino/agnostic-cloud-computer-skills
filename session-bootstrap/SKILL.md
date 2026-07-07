---
name: session-bootstrap
description: Session start checklist for agents on any cloud computer. Ensures context is loaded, repos are current, and scope is confirmed before any work begins.
author: Gerardo Treviño
created: 2026-06-27
---

# Session Bootstrap Checklist

Run these steps at the start of every session, before taking any execution steps.

## 1. Read Machine Context

```bash
cat ~/AGENTS.md
```

This file contains machine-specific rules, running projects, port allocations, and installed tools. Read it completely.

## 2. Load MemPalace Context

```bash
export PATH="$HOME/.local/bin:$PATH"
mempalace wake-up
```

This loads the L0 (identity) and L1 (essential story) context layers — approximately 600–900 tokens summarizing the current state of all projects on this machine.

## 3. Pull Latest Skills

```bash
cd ~/skills && git pull --rebase origin main
mempalace mine ~/skills/
```

Always work from the latest version of the operating contract and all skills.

## 4. Pull Latest Project Code

For the active project:

```bash
cd ~/projects/<project-name>
git pull --rebase origin main
```

Never begin work on a stale or behind branch.

## 5. Check Running Containers

```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

Know what is running before making changes.

## 6. Confirm Scope

State the task clearly to the user and confirm understanding before writing any code or making any changes.

---

## End-of-Session Protocol

Before closing the session:

1. Write `docs/sessions/YYYY-MM-DD.md` in the active project — what was done, what decisions were made, what is incomplete, what the next step is.
2. Commit and push all `docs/` changes including the session log.
3. Run `mempalace mine ~/projects/<project-name>` to index the changes.
4. Summarize what was accomplished and what the next step is.
