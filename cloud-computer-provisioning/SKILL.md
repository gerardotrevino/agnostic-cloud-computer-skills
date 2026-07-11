# Cloud Computer Provisioning Skill

**Author:** Manus AI
**Organization:** Paybook, Inc.

This skill provides the step-by-step procedure for provisioning a new persistent cloud computer (Ubuntu VM) for a Paybook developer. Any agent tasked with setting up a new machine MUST follow this exact sequence to ensure consistency, security, and compliance with the operating contract.

## Prerequisites

Before executing any commands on the new machine, you MUST collect the developer's identity. If the user has not provided this information, use the `ask` tool to request it:

1. **Name** (e.g., "Gerardo Treviño")
2. **Organization / Role** (e.g., "Paybook, Inc.")
3. **Domain / Subdomain** (e.g., `g-cc-1.internal-services.network`)

Do not proceed until you have the domain. It is required for the Caddy TLS setup.

## Phase 1: System Basics & Docker

1. **Update system packages:**
   ```bash
   sudo apt-get update && sudo apt-get upgrade -y
   sudo apt-get install -y curl wget git jq unzip make lsof net-tools
   ```

2. **Install Docker & Docker Compose:**
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   sudo usermod -aG docker ubuntu
   ```
   *(Note: The agent may need to use `sudo` for docker commands initially, or restart the shell session for the group change to take effect).*

## Phase 2: Shared Services & Network

1. **Create the shared Docker network:**
   ```bash
   docker network create dev-shared
   ```

2. **Clone the shared-services repo (public, no PAT needed):**
   ```bash
   mkdir -p ~/projects
   cd ~/projects
   git clone https://github.com/gerardotrevino/cloud-computer-shared-services.git shared-services
   ```

3. **Configure the developer's domain:**
   Create the `.env` file (this is gitignored — machine-specific):
   ```bash
   cd ~/projects/shared-services
   echo "DEV_DOMAIN=<actual-domain>" > .env
   ```
   Replace `<actual-domain>` with the domain the developer provided (e.g., `r-cc-1.internal-services.network`). The Caddyfile reads `{$DEV_DOMAIN}` from this file automatically — do NOT edit the Caddyfile itself.

4. **Start shared services:**
   ```bash
   cd ~/projects/shared-services
   docker compose up -d
   ```
   Verify that Postgres, Redis, Caddy, and Adminer are running. Check Caddy logs:
   ```bash
   docker logs shared-caddy 2>&1 | grep -i "error\|domain\|certificate"
   ```
   You should see `enabling automatic TLS certificate management` with the correct domain.

## Phase 3: Firewall & Ports

Configure UFW (Uncomplicated Firewall) to allow traffic. **Do not enable UFW blindly without allowing SSH (port 22) first, or you will lock everyone out.**

**IMPORTANT:** We use **port-per-project routing**, NOT subdomain routing. Caddy listens on EACH project's external port (e.g., 8000, 8001, 8002, 8060, 8070) and serves HTTPS directly on those ports. These are NOT raw application ports — they are Caddy-managed HTTPS endpoints. They MUST be opened on UFW.

The architecture is:
- External (internet) → `https://domain:8000` → Caddy (HTTPS, port 8000) → `127.0.0.1:18000` (app, localhost only)
- The app binds to `127.0.0.1:(PORT+10000)` — NEVER exposed to the internet
- Caddy binds to `0.0.0.0:PORT` — this is the public HTTPS endpoint

Therefore, open ALL ports that Caddy will serve on:

```bash
sudo ufw allow 22/tcp     # SSH
sudo ufw allow 80/tcp     # Caddy HTTP→HTTPS redirect + ACME challenge
sudo ufw allow 443/tcp    # Caddy default HTTPS (if needed)
sudo ufw allow 5050/tcp   # Adminer (via Caddy HTTPS)
```

**You MUST also open every active project port block.** Check `~/projects/PORT_REGISTRY.md` for which blocks are in use. For example:
```bash
sudo ufw allow 8000:8009/tcp   # Block 01 — Paybook Finances
sudo ufw allow 8060:8069/tcp   # Block 07 — Paybook UI
sudo ufw allow 8070:8079/tcp   # Block 08 — Syncfy 2.0
```

Do NOT skip project ports thinking they should be "private." The private ports are the +10000 offset ones (18000, 18060, 18070) which are bound to localhost and never need UFW rules. The external ports (8000, 8060, 8070) are Caddy HTTPS endpoints and MUST be publicly accessible.

Enable the firewall:
```bash
sudo ufw --force enable
```

## Phase 4: MemPalace & Skills

1. **Install `uv` and `mempalace`:**
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   source $HOME/.cargo/env
   uv tool install mempalace
   ```

2. **Configure Identity:**
   ```bash
   mkdir -p ~/.mempalace
   echo "<Developer Name> / <Organization>" > ~/.mempalace/identity.txt
   ```

3. **Clone the Skills repo:**
   ```bash
   cd ~
   git clone https://github.com/Paybook/skills.git
   ```

## Phase 5: Verification & Handoff

1. **Verify DNS:** Run `nslookup <domain> 8.8.8.8` to ensure the domain points to the machine's IP. If it returns NXDOMAIN, inform the user that they must create the A record before Caddy can provision the Let's Encrypt certificate.
2. **Verify Caddy logs:** `docker logs shared-caddy` to ensure no fatal errors.
3. **Verify Adminer:** Ensure `https://<domain>:5050` is accessible (once DNS is active).

Deliver a final report to the user confirming the machine is ready, the shared services are running, and MemPalace is initialized.
