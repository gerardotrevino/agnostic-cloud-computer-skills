---
name: session-bootstrap
description: Session initialization and termination protocol. Implements contract Sections 3 (Session Start) and 19 (End of Session). Read this skill at the start of every session.
metadata:
  author: Gerardo Treviño Rojas
  contributors: []
  created: 2026-06-25
  version: "3.0"
---

# Session Bootstrap

This skill is the executable implementation of the operating contract's Session Start Protocol (Section 3) and End of Session Protocol (Section 19). It applies to every session on every machine, regardless of project or discipline.

**This skill is mandated by the operating contract Section 17.1 (Global Mandatory Skills).**

---

## Session Start Protocol (8 Steps)

Complete these steps in order at the beginning of every session. Do not begin implementation until all eight steps are complete.

---

### Step 1: Read Machine Context

Read `~/AGENTS.md` on the cloud computer. This file contains:
- Machine-specific configuration (ports, installed tools, running services)
- Project-specific notes and session history
- System-level changes log

If `~/AGENTS.md` does not exist, the machine has not been properly set up. Use the `cloud-computer-setup` skill to provision it.

---

### Step 2: Verify Identity

Read `~/.mempalace/identity.txt`. Check that it contains all required fields:

```
name: [Full Name]
email: [Email]
github: [GitHub username]
role: [Position/Title]
org: [Organization name]
```

**If the file exists and is complete:** Identity is confirmed. Proceed without asking. Do not re-derive from the PAT. Do not ask "who am I working with?" on a single-developer machine.

**If the file does not exist or is incomplete:** Follow contract Section 18 to establish identity. Ask the developer for their full name, email, role, and organization before proceeding with any work.

---

### Step 3: Load Memory

```bash
export PATH="$HOME/.local/bin:$PATH"
mempalace wake-up
```

This loads the L0 (identity) and L1 (essential story) context. Review the output to understand what projects exist and what was done in recent sessions.

---

### Step 4: Sync Skills

Pull the latest skills from the skills repository:

```bash
cd ~/skills && git pull --rebase origin main
```

If the machine has a secondary skills repo (e.g., `~/org-skills`), pull that too:

```bash
cd ~/org-skills && git pull --rebase origin main 2>/dev/null || true
```

**If no PAT is configured and pull fails:** Ask the developer for a GitHub PAT with read access to the skills repo. Store it in `~/.git-credentials` (mode 600).

---

### Step 5: Read Mandatory Skills

Read all **global mandatory skills** (contract Section 17.1):

| Skill | Path | When |
|-------|------|------|
| `software-engineering-standards` | `~/skills/software-engineering-standards/SKILL.md` | Before any development work |

Then read the **project's `AGENTS.md`** and read all skills listed in its **Required Skills** section.

---

### Step 6: Sync Project

```bash
cd ~/projects/<active-project>
git fetch origin
git status
```

**If clean (no local changes):**
```bash
git pull --rebase origin main
```

**If uncommitted changes or open branches exist from a previous session:**
- Do NOT pull automatically
- Ask the developer: "There are uncommitted changes / an open branch from a previous session. Should I commit and push them, stash them, or discard them?"
- Resolve before proceeding

**If on a feature branch:**
- Stay on the feature branch unless the developer says otherwise
- Pull the latest from the feature branch's remote tracking branch

---

### Step 7: Verify Environment

```bash
cd ~/projects/<active-project>
docker compose ps
```

**If containers are running and healthy:** Proceed.

**If containers are not running:**
```bash
docker compose up -d
```
Wait for all containers to be healthy. If any container fails to start, diagnose and fix before proceeding.

**If the project has a database, check for pending migrations:**
- Look for a `Makefile` target like `make migrate` or a migrations directory
- If new migration files exist that have not been applied, apply them
- If unsure, ask the developer

**If Docker daemon is not running:**
```bash
sudo systemctl start docker
```

---

### Step 8: Confirm Scope

Ask the developer what they want to accomplish in this session. Do not assume. Do not start working on something from a previous session without confirmation.

> "Environment is ready. What would you like to work on today?"

---

## End of Session Protocol (7 Steps)

Complete these steps at the end of every session — whether the work is complete or interrupted. Do not end a session without completing this protocol.

---

### Step 1: Commit All Changes

All modified files must be committed with a descriptive commit message following Conventional Commits format:

```bash
git add -A
git status  # review what's being committed
git commit -m "<type>(<scope>): <description>"
```

Do not leave uncommitted work. If work is incomplete, commit it on the feature branch with a clear message indicating it is work-in-progress:

```bash
git commit -m "wip(<scope>): <description> — incomplete, continuing next session"
```

---

### Step 2: Push to Remote

```bash
git push origin <branch-name>
```

All work must be pushed. Local-only commits are at risk of being lost.

---

### Step 3: Write Session Log

Create or append to the session log at `docs/sessions/YYYY-MM-DD.md`:

```markdown
## Session: YYYY-MM-DD HH:MM (timezone)

**Developer:** [Full name from identity file]
**Duration:** [Approximate duration]
**Branch:** [Branch name]
**Commits:** [First commit SHA..Last commit SHA]

### What Was Done
[Bullet list of concrete accomplishments — not intentions, not plans, but what was actually completed and verified.]

### Decisions Made
[Any architectural, design, or implementation decisions made during this session. Reference ADRs if created.]

### Issues Encountered
[Problems hit during the session and how they were resolved. If unresolved, state clearly.]

### Next Steps
[What should happen in the next session. Be specific — not "continue working" but "implement the payment webhook handler and write integration tests for it."]
```

**Rules:**
- Session logs must be factual. Do not log intentions — log results.
- If nothing was accomplished (session was all research/planning), log that honestly.
- Commit the session log as part of the final commit.

---

### Step 4: Update Project AGENTS.md

Update the project-level `AGENTS.md` (if it exists) with:
- Current state of the project
- What changed in this session
- Any new dependencies, services, or configuration changes

---

### Step 5: Mine MemPalace

```bash
export PATH="$HOME/.local/bin:$PATH"
mempalace mine ~/projects/<project-name>
```

This indexes all new and modified files into the palace for future sessions.

---

### Step 6: Mine Skills (If Updated)

If any skills were created or modified during the session:

```bash
mempalace mine ~/skills/
```

---

### Step 7: Verify Push

Confirm the push succeeded:

```bash
git log --oneline -3
git status
```

If the project has CI, check that it is running:

```bash
gh run list --limit 1 2>/dev/null || true
```

Do not end the session with a failed push or broken CI.

---

## Git Branch Protocol

All development work happens on feature branches. Direct pushes to `main` are prohibited.

### Creating a Branch

```bash
git checkout main
git pull --rebase origin main
git checkout -b <type>/<short-description>
```

Branch naming follows the same types as commit messages:

| Type | Example |
|------|---------|
| `feat/` | `feat/user-authentication` |
| `fix/` | `fix/pagination-off-by-one` |
| `docs/` | `docs/update-api-reference` |
| `refactor/` | `refactor/extract-payment-service` |
| `chore/` | `chore/upgrade-dependencies` |

### Merging to Main

1. Push the feature branch
2. Verify CI passes on the branch
3. Merge to main (squash merge preferred for clean history):
   ```bash
   git checkout main
   git pull --rebase origin main
   git merge --squash <branch-name>
   git commit -m "<type>(<scope>): <description>"
   git push origin main
   ```
4. Delete the feature branch:
   ```bash
   git branch -d <branch-name>
   git push origin --delete <branch-name>
   ```

### Override for Solo Sessions

If the developer explicitly requests to work directly on `main` for a specific session (e.g., documentation-only changes, trivial fixes), they may override the branch requirement verbally. The agent must confirm: "You want to commit directly to main for this session — confirmed?" This override applies only to the current session and must be noted in the session log.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `mempalace` not found | `export PATH="$HOME/.local/bin:$PATH"` or reinstall: `uv tool install mempalace` |
| Docker daemon not running | `sudo systemctl start docker && sudo systemctl enable docker` |
| Git pull fails (no PAT) | Ask developer for PAT, store in `~/.git-credentials` (mode 600) |
| Identity file missing | Follow contract Section 18 — ask developer, create the file |
| Containers unhealthy | Check logs: `docker compose logs <service>` — fix before proceeding |
| Stale branch from previous session | Ask developer how to proceed — do not auto-merge or discard |
| CI failing | Do not merge to main. Fix the issue on the feature branch first. |
| Port conflict | Check `~/projects/PORT_REGISTRY.md` — stop conflicting service or reassign |
| Skills repo diverged | `git pull --rebase origin main` — resolve conflicts, never force-push |
