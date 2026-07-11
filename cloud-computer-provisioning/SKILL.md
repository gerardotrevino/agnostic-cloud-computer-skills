# Cloud Computer Provisioning Skill

**Author:** Manus AI
**Organization:** 

This skill provides the step-by-step procedure for provisioning a new persistent cloud computer (Ubuntu VM) for a Paybook developer. Any agent tasked with setting up a new machine MUST follow this exact sequence to ensure consistency, security, and compliance with the operating contract.

## Prerequisites

Before executing any commands on the new machine, you MUST collect the developer's identity. If the user has not provided this information, use the `ask` tool to request it:

1. **Name** (e.g., "Gerardo Treviño")
2. **Organization / Role** (e.g., "")
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

2. **Clone the dev-infrastructure repo:**
   ```bash
   mkdir -p ~/projects
   cd ~/projects
   git clone https://github.com/Paybook/dev-infrastructure.git shared-services
   ```
   *(If a PAT is needed and not in `~/.git-credentials`, ask the user).*

3. **Configure Caddy with the developer's domain:**
   Read `~/projects/shared-services/Caddyfile`. Replace all instances of the example domain with the actual domain provided by the developer.
   ```bash
   sed -i 's/example-domain.network/<actual-domain>/g' ~/projects/shared-services/Caddyfile
   ```

4. **Start shared services:**
   ```bash
   cd ~/projects/shared-services
   docker compose up -d
   ```
   Verify that Postgres, Redis, Caddy, and Adminer are running.

## Phase 3: Firewall & Ports

Configure UFW (Uncomplicated Firewall) to allow traffic. **Do not enable UFW blindly without allowing SSH (port 22) first, or you will lock everyone out.**

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 5050/tcp  # Adminer
```

If the developer is assigned specific project port blocks (e.g., Block 01: 8000-8009), open those as well:
```bash
sudo ufw allow 8000:8009/tcp
```

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
