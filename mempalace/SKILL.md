---
name: mempalace
description: Complete guide for MemPalace — persistent semantic memory for cloud computers. Covers installation, identity, mining, searching, the three-layer context strategy, proactive querying, health verification, troubleshooting, and disaster recovery.
author: Gerardo Treviño Rojas
created: 2026-06-27
updated: 2026-07-18
version: "2.0"
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
name: [Your Name]
email: [Your Email]
github: [GitHub Username]
role: [Your Role]
org: [Organization Name]
machine: [Device ID or hostname]
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
| `mempalace repair-status` | Diagnose palace health (HNSW vs SQLite divergence) |
| `mempalace repair --mode from-sqlite --backup --archive-existing` | Full rebuild from SQLite ground truth |

## The Three-Layer Context Strategy

**Layer 1 — Git (durable, shared):** All project context lives in `docs/` committed to Git. This is the source of truth. Any developer on any machine gets full context on `git clone`.

**Layer 2 — MemPalace (semantic, local):** A semantic index over project files. Answers *why* decisions were made. Always rebuildable from Git. If the machine is wiped, clone the repo and run `mempalace mine` to fully restore context in minutes.

**Layer 3 — CodeGraph (structural, local):** A structural index over source code. Answers *what* calls what and what the blast radius of a change is. Rebuilt instantly with `codegraph sync`.

The palace is **never committed to Git** — it is local and machine-specific. Context sync between developers happens through Git, not through the palace directly.

## Proactive Context Query (Mandatory)

Before answering any question where the agent lacks full confidence:

1. Run `mempalace search "<relevant query>"` first
2. If the answer exists — use it, cite the source
3. If not found — research, implement, document in `docs/`, then mine immediately

**The compounding knowledge rule:** Every session must leave the palace richer than it found it. An agent that discovers knowledge and does not persist it is destroying organizational context.

## Mining Protocol

### End-of-Session Mining (mandatory)

```bash
# After pushing docs/ changes to Git:
mempalace mine ~/projects/<project-name>
```

### Immediate Mining (on knowledge capture)

Mine immediately (don't wait for session end) when:
- A bug is diagnosed and documented
- A new tool/service is installed and documented
- A design decision is made (ADR written)
- A failure mode is discovered and documented
- A workflow is established

```bash
# Mine right after documenting new knowledge
mempalace mine ~/projects/<project-name>
```

## Health Verification (Session Start)

After `mempalace wake-up`, verify writes work:

```bash
mempalace status   # note the drawer count
mempalace mine ~/projects/<project-name>
mempalace status   # count should increase (or stay same if no new files)
```

If count doesn't change when new files exist → the palace has a write corruption issue. **Do not proceed.** Fix it first.

## Troubleshooting & Recovery

### Symptom: Silent Write Failures

**How it manifests:**
- `mempalace mine` reports "Drawers filed: N" (success) but `mempalace status` shows no increase in drawer count
- New content is not findable via `mempalace search`
- No error messages — completely silent

**Root cause (discovered 2026-07-18):**
ChromaDB's HNSW vector index can become corrupted in two ways:
1. **Poisoned watermark:** The `max_seq_id` in the metadata table gets ahead of actual data, causing ChromaDB to discard all new writes as "already applied"
2. **HNSW header corruption:** The `header.bin` file in the HNSW segment gets `max_elements=1`, capping the index and silently dropping inserts

Both failures are completely silent — no errors, no warnings. Reads continue to work for old data.

### Diagnosis

```bash
# Step 1: Check repair-status
mempalace repair-status
# Look for: "HNSW divergence" or "watermark mismatch"

# Step 2: Verify with a test write
mempalace mine ~/projects/<any-project>
mempalace status
# If count didn't change → writes are broken
```

### Recovery: Migration Approach (Recommended for Large Palaces)

The built-in `mempalace repair --mode from-sqlite` may fail on large palaces (>40K drawers) due to memory constraints. The reliable fix is a batch migration to a fresh palace:

```python
import chromadb
import os

# 1. Rename the corrupt palace
os.rename(os.path.expanduser("~/.mempalace/palace"),
          os.path.expanduser("~/.mempalace/palace_corrupt"))

# 2. Create fresh palace
os.makedirs(os.path.expanduser("~/.mempalace/palace"), exist_ok=True)

# 3. Open both
old_client = chromadb.PersistentClient(
    path=os.path.expanduser("~/.mempalace/palace_corrupt"))
new_client = chromadb.PersistentClient(
    path=os.path.expanduser("~/.mempalace/palace"))

# 4. For each collection, batch-copy in chunks of 500
for col_info in old_client.list_collections():
    old_col = old_client.get_collection(col_info.name)
    new_col = new_client.get_or_create_collection(
        name=col_info.name,
        metadata=col_info.metadata)
    
    all_ids = old_col.get(include=[])['ids']
    batch_size = 500
    
    for i in range(0, len(all_ids), batch_size):
        batch_ids = all_ids[i:i+batch_size]
        batch = old_col.get(
            ids=batch_ids,
            include=['embeddings', 'documents', 'metadatas'])
        new_col.add(
            ids=batch['ids'],
            embeddings=batch['embeddings'],
            documents=batch['documents'],
            metadatas=batch['metadatas'])

print(f"Migration complete. New count: {new_col.count()}")
```

### Recovery: Quick Fix (Small Divergence ≤ 10 drawers)

If `repair-status` shows only a small HNSW divergence (e.g., 8 drawers):

```bash
# Try the built-in repair
mempalace repair --mode from-sqlite --backup --archive-existing
```

This works for small palaces (<10K drawers) or small divergences. For large palaces, use the migration approach above.

### After Recovery

```bash
# Verify writes work
mempalace status
mempalace mine ~/projects/<any-project>
mempalace status  # count should increase

# Re-mine ALL projects to catch up on lost content
for dir in ~/projects/*/; do
  mempalace mine "$dir"
done
mempalace mine ~/skills/
```

### Prevention

- Always verify writes work at session start (health check above)
- If the machine crashes or Docker kills a process mid-write, check palace health immediately in the next session
- Keep the corrupt palace backup until you've verified the new one is healthy
- Consider periodic backups: `cp ~/.mempalace/palace/chroma.sqlite3 ~/.mempalace/palace/chroma.sqlite3.bak`
