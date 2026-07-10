---
name: cloud-computer-setup
description: Complete guide for agents to provision a new cloud computer with Docker, MemPalace, and a multi-project architecture. Agnostic instructions — works for any team, organization, or project type.
metadata:
  author: Gerardo Treviño Rojas
  contributors: []
  created: 2026-06-27
  version: "2.2"
---

# Cloud Computer Setup Protocol

## Overview

This skill provides the exact step-by-step protocol for an agent to set up a new persistent cloud computer (Ubuntu VM). It establishes a **multi-project Docker Compose architecture** where every project is **fully self-contained** — each project defines its own database, cache, and services inside its own `docker-compose.yml`. No project depends on any shared host-level service.

The **instructions in this skill are fully agnostic** — they contain no organization-specific details. Team-specific configuration (GitHub org, skills repo URL, developer identity, machine IP) is collected from the user at setup time and written into the machine's own `AGENTS.md`.

**When to use this skill:**
- A raw or fresh cloud computer is attached to the session.
- The user asks to "set up the developer environment on this machine".
- You need to replicate a persistent multi-project dev environment on a new node.

---

## Before You Start

Collect the following from the user before running any commands. These values will be written into `AGENTS.md` and `~/.mempalace/identity.txt` on the machine — not hardcoded into any shared skill or script.

| Item | Purpose |
|------|---------|
| Developer name | MemPalace identity, Git author |
| Organization name | MemPalace identity |
| Developer role | MemPalace identity |
| Git email | Git author (repo-local and global) |
| Machine external IP | Used in README and access URLs |
| GitHub skills repo URL (optional) | To pull shared skills onto the machine |
| GitHub PAT (optional) | To authenticate private repos |

---

## Phase 1: System Provisioning

Run these commands on the remote machine to install core dependencies.

```bash
# Update system and install core tools
sudo apt-get update -qq
sudo apt-get install -y make jq git htop tree curl wget software-properties-common openssl bc net-tools

# Install Docker & Docker Compose (skip if already present)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu
sudo systemctl enable docker
rm get-docker.sh

# Verify Docker daemon is running and will start on reboot
sudo systemctl start docker
docker --version
docker compose version

# Install Python 3.12, pip, and uv
sudo apt-get install -y python3.12 python3-pip python3.12-venv
curl -LsSf https://astral.sh/uv/install.sh | sh
export PATH="$HOME/.local/bin:$PATH"

# Persist PATH additions in .bashrc
cat >> ~/.bashrc << 'BASHRC'
export PATH="$HOME/.local/bin:$PATH"
BASHRC
```

### Optional: Go Toolchain

Required only if Go-based projects will be developed on this machine.

```bash
# Install Go (latest stable)
GO_VERSION="1.22.5"  # or latest; projects use GOTOOLCHAIN=auto to self-resolve
wget -q "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz"
sudo tar -C /usr/local -xzf "go${GO_VERSION}.linux-amd64.tar.gz"
rm "go${GO_VERSION}.linux-amd64.tar.gz"

# Add Go to PATH
cat >> ~/.bashrc << 'BASHRC'
export PATH="$PATH:/usr/local/go/bin:$HOME/go/bin"
BASHRC
export PATH="$PATH:/usr/local/go/bin:$HOME/go/bin"
```

### Optional: mise (polyglot version manager)

Useful when projects pin specific tool versions via `.mise.toml` or `.tool-versions`.

```bash
curl https://mise.run | sh
# Binary lands at ~/.local/bin/mise — no shell hook needed for CI-style usage
```

---

## Phase 2: MemPalace Installation

MemPalace is the critical memory layer. Install it officially via `uv`.

```bash
# Install MemPalace
uv tool install mempalace

# Configure identity — replace with real values from user
mkdir -p ~/.mempalace
cat > ~/.mempalace/identity.txt << 'EOF'
name: [Developer Name]
email: [Developer Email]
github: [GitHub Username]
role: [Developer Role]
org: [Organization Name]
machine: [Device ID or hostname]
EOF

# Initialize the dev_environment wing
mkdir -p ~/projects
mempalace init ~/projects --yes --no-llm --auto-mine
```

---

## Phase 3: Project Architecture Scaffolding

Create the standard directory structure and templates for multi-project hosting.

```bash
# Create directory structure
mkdir -p ~/projects/_templates/{node-app,python-app,static-site,go-api}

# Create the shared Docker network (optional, for cross-project communication if needed)
docker network create dev-shared || true
```

### 3.1 Port Registry

The port registry prevents conflicts across projects. Each project claims a block of 10 ports. Create `~/projects/PORT_REGISTRY.md`:

```markdown
# Port Registry

Rules:
1. Claim a block before starting a new project.
2. The scaffolding script (`new-project.sh`) updates this file automatically.
3. Open ports in UFW only when external access is required: `sudo ufw allow <port>/tcp`

| Block     | Port Range  | Project Name      | Status      | UFW Open? |
|-----------|-------------|-------------------|-------------|-----------|
| Block 01  | 8000–8009   | (available)       | Available   | No        |
| Block 02  | 8010–8019   | (available)       | Available   | No        |
| Block 03  | 8020–8029   | (available)       | Available   | No        |
| Block 04  | 8030–8039   | (available)       | Available   | No        |
| Block 05  | 8040–8049   | (available)       | Available   | No        |
| Block 06  | 8050–8059   | (available)       | Available   | No        |
| Block 07  | 8060–8069   | (available)       | Available   | No        |
| Block 08  | 8070–8079   | (available)       | Available   | No        |
| Block 09  | 8080–8089   | (available)       | Available   | No        |
| Block 10  | 8090–8099   | (available)       | Available   | No        |
| Block 11  | 8100–8109   | (available)       | Available   | No        |
| Block 12  | 8110–8119   | (available)       | Available   | No        |
| Block 13  | 8120–8129   | (available)       | Available   | No        |
| Block 14  | 8130–8139   | (available)       | Available   | No        |
| Block 15  | 8140–8149   | (available)       | Available   | No        |
| Block 16  | 8150–8159   | (available)       | Available   | No        |
| Block 17  | 8160–8169   | (available)       | Available   | No        |
| Block 18  | 8170–8179   | (available)       | Available   | No        |
| Block 19  | 8180–8189   | (available)       | Available   | No        |
| Block 20  | 8190–8199   | (available)       | Available   | No        |
```

### 3.2 Global Makefile

Create `~/projects/Makefile` for discoverability and common operations:

```makefile
.PHONY: status help new-project

## Show status of all running containers
status:
	@docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
	@echo ""
	@sudo ufw status numbered

## Create a new project (usage: make new-project NAME=myapp TEMPLATE=node-app PORT=8000)
new-project:
	@./new-project.sh $(NAME) $(TEMPLATE) $(PORT)

## Show this help
help:
	@echo "Targets: status | new-project | help"
	@echo "Templates: node-app | python-app | static-site | go-api"
	@echo "Example: make new-project NAME=my-api TEMPLATE=go-api PORT=8020"
```

### 3.3 Scaffolding Script

Create `~/projects/new-project.sh` (chmod +x). This script **enforces** the multi-project contract — it does not merely print a manual checklist. When invoked, it must perform all of the following automatically:

1. Validate the requested port block is `Available` in `PORT_REGISTRY.md`.
2. Copy the template into `~/projects/<name>/`.
3. Generate strong random credentials for the project's own database.
4. Write all DB credentials (`DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`) into the project's `.env` (gitignored).
5. Claim the port block in `PORT_REGISTRY.md` (rewrite the row to `Active`).
6. Generate the full `docs/` knowledge base (see Phase 5), including `docs/PROJECT_RULES.md`.
7. Initialize a Git repository with a first commit.
8. Run `mempalace init <dir> --yes --no-llm --auto-mine` under a timeout (max 120s) to guarantee a memory wing. If it times out, the project is still complete; memory can be rebuilt later.

The only remaining manual step is `sudo ufw allow <port>/tcp` (host firewall change).

Usage: `./new-project.sh <project-name> <template> <port>`

### 3.4 Templates

**CRITICAL: Every template MUST be fully self-contained.**
The `docker-compose.yml` for every template must define its own database, cache, and application services. No project depends on external shared services.

Each template must include a `.env.example` documenting the expected variables.

**Available templates:**
| Template | Stack | Use Case |
|----------|-------|----------|
| `node-app` | Node.js + TypeScript | Web apps, APIs, tRPC services |
| `go-api` | Go + chi/connect | High-performance APIs, microservices |
| `python-app` | Python 3.12 + FastAPI | ML services, data pipelines, scripts |
| `static-site` | nginx | Landing pages, documentation sites |

---

## Phase 4: Git Credential Setup

Configure Git globally so all repos on the machine authenticate via the credential store.

```bash
# Set global Git identity
git config --global user.name "[Developer Name]"
git config --global user.email "[Git Email]"
git config --global credential.helper store
git config --global init.defaultBranch main

# Write the PAT to the credential store (mode 600 for security)
cat > ~/.git-credentials << 'EOF'
https://[org-or-user]:[PAT]@github.com
EOF
chmod 600 ~/.git-credentials
```

**Security:** The PAT must NEVER appear in any repo's `.git/config` remote URL. Always use `https://github.com/<org>/<repo>.git` as the remote and let the credential store supply auth transparently.

---

## Phase 5: AGENTS.md

Create `~/AGENTS.md` — this file is auto-read by agents at the start of every session. It contains machine-specific operational rules. Fill in the values collected from the user in "Before You Start".

```markdown
# Agent Instructions for Cloud Computer ([device-id])

## System Overview
This is a persistent developer environment designed for running multiple projects simultaneously using Docker Compose.

**Core Principles:**
1. **No Port Conflicts:** Every project MUST use a pre-allocated port block.
2. **Self-Contained Projects:** Each project bundles its own database and services via Docker Compose. No shared-services dependency for application databases.
3. **Documentation First:** All changes MUST be recorded in the `mempalace` skill.
4. **Git Sync:** Agents MUST remind the user to push code to their GitHub repos to prevent context loss.

## Directory Structure
- `~/projects/` — Root for all developer work.
- `~/projects/PORT_REGISTRY.md` — **MUST READ THIS** before starting any new project.
- `~/projects/new-project.sh` — Script to scaffold new projects.
- `~/projects/Makefile` — Global commands (`make status`).

## Creating a New Project
1. Read `~/projects/PORT_REGISTRY.md` to find an available port block.
2. Run `./new-project.sh <name> <template> <port>` in `~/projects/`.
3. Open the UFW port if external access needed: `sudo ufw allow <port>/tcp`.
4. Link to GitHub remote and push the initial commit.

## docs/ Knowledge Base Convention
Every project MUST have a `docs/` directory committed to Git.
- `docs/README.md` — entry point
- `docs/PRD.md` — Product Requirements
- `docs/architecture.md` — system architecture
- `docs/api-reference.md` — API endpoints
- `docs/data-models.md` — database schema
- `docs/decisions/` — ADRs
- `docs/sessions/` — daily session logs

## Managing the MemPalace
MemPalace is installed at `~/.local/bin/mempalace` (via `uv tool install mempalace`).
- Start of session: `mempalace wake-up`
- End of session: `mempalace mine ~/projects/<project-name>`
- Search: `mempalace search "query"`

## Git Protocol
- Always suggest committing and pushing after significant changes.
- GitHub PAT is configured in `~/.git-credentials` (mode 600).

## Skills Available on This Machine
| Skill | Path | Purpose |
|-------|------|---------|
| (populated after Phase 6) | | |

## System-Level Changes Log
| Date | Change | Details |
|------|--------|---------|
| [setup date] | Initial provisioning | cloud-computer-setup skill v2.2 |
```

---

## Phase 6: docs/ Knowledge Base Convention

Every project MUST include a `docs/` directory committed to Git. This is the shared context layer — all developers and agents receive full project knowledge the moment they clone the repo.

The `new-project.sh` script creates this structure automatically for every project:

```
project-name/
└── docs/
    ├── README.md              ← entry point, conventions, onboarding guide
    ├── PRD.md                 ← Product Requirements Document
    ├── architecture.md        ← system architecture and tech stack
    ├── api-reference.md       ← API endpoint reference
    ├── data-models.md         ← database schema and entity relationships
    ├── PROJECT_RULES.md       ← project-specific rules (inherits machine-wide)
    ├── decisions/             ← Architecture Decision Records (ADRs)
    │   └── 000-adr-template.md
    └── sessions/              ← daily session logs
        └── YYYY-MM-DD.md
```

**The two-layer context strategy:**

- **Layer 1 (Git):** All context documents live in `docs/` and are committed. This is the durable, portable, shared source of truth. Any developer on any machine gets full project knowledge on `git clone`.
- **Layer 2 (MemPalace):** The fast semantic search index over Layer 1. If the machine is wiped, clone the repo and run `mempalace mine` to fully restore context in minutes. The palace is always rebuildable from Git.

---

## Phase 7: Skills Installation

Skills are agnostic, reusable modules that agents read to understand how to perform specific tasks. **The correct model is a real Git clone** (not a copy that strips `.git`), so edits can be committed and pushed back.

### Layer 1: Agnostic Skills (public, no PAT required)

The agnostic skills repo contains the operating contract, setup guides, session bootstrap, and all general-purpose skills. It is public and works for any team or individual.

```bash
# Clone the agnostic skills repo
cd ~ && git clone https://github.com/gerardotrevino/agnostic-cloud-computer-skills.git ~/skills

# Set repo-local identity
cd ~/skills && git config user.name "[git-username]" && git config user.email "[git-email]"

# Mine skills into the palace
export PATH="$HOME/.local/bin:$PATH"
mempalace mine ~/skills/
mempalace mine ~/AGENTS.md
```

### Layer 2: Organization-Specific Skills (private, optional)

If the team has a private skills repo containing internal tooling, proprietary workflows, or sensitive context, clone it alongside the agnostic skills:

```bash
# Requires a PAT in ~/.git-credentials for the private org
git clone https://github.com/<org>/skills.git ~/org-skills
cd ~/org-skills && git config user.name "[git-username]" && git config user.email "[git-email]"
mempalace mine ~/org-skills/
```

Do not merge repos — keep them separate for access control. The agnostic skills in `~/skills` are always the base layer. Org-specific skills in `~/org-skills` extend or override them for that team's context.

---

## Phase 8: UFW Firewall Hardening

Configure the firewall to deny all inbound by default, allowing only SSH.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw --force enable
sudo ufw status
```

Ports are opened per-project as needed (`sudo ufw allow <port>/tcp`) and closed when no longer required. The port registry tracks which are open.

---

## Phase 9: Verification

Confirm the environment is fully operational before handing off to the user:

```bash
docker ps                          # running containers
sudo ufw status                    # firewall status
export PATH="$HOME/.local/bin:$PATH"
mempalace wake-up                  # MemPalace working
mempalace status                   # palace wings and drawer count
ls ~/skills/                       # skills installed
cat ~/projects/PORT_REGISTRY.md    # port registry present
```

Deliver a summary to the user covering: installed tools and versions, running services and ports, palace status (wings and drawer count), and next steps for creating the first project.

---

## Quick Reference: Creating a Project After Setup

```bash
cd ~/projects
./new-project.sh my-api go-api 8020
# Script handles: template copy, DB provisioning, port claim, docs/, git init, mempalace
sudo ufw allow 8020/tcp  # only if external access needed
git remote add origin https://github.com/<org>/my-api.git
git push -u origin main
```
