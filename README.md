# agnostic-cloud-computer-skills

A collection of fully agnostic agent skills for setting up and operating persistent cloud computers with a multi-project Docker Compose architecture.

These skills contain no organization-specific content. They work for any team, any project, and any developer. Organization-specific values (team name, GitHub org, skills repo URL, developer identity) are collected at setup time and written into the machine's own `AGENTS.md`.

## Skills in This Repo

| Skill | Purpose |
|-------|---------|
| `cloud-computer-setup` | Provision a new cloud computer: Docker, MemPalace, project scaffolding, Git credentials, UFW |
| `agent-operating-contract` | Master operating contract governing all agent behavior on any project |
| `session-bootstrap` | Session start checklist: wake-up, pull, confirm scope |
| `prd-rfp` | Write Product Requirements Documents and technical specs before building |
| `software-engineering-standards` | Coding standards, ADRs, testing, CI/CD conventions |
| `skill-creator` | Create and update skills in this format |
| `internet-skill-finder` | Find relevant skills from GitHub for new domains |
| `infra-onboarding` | First-session guide for an infrastructure agent on a new org's cloud computer |

## How to Use

On a new cloud computer, pull this repo and read `cloud-computer-setup/SKILL.md` to begin provisioning:

```bash
git clone https://github.com/gerardotrevino/agnostic-cloud-computer-skills.git ~/skills
```

No PAT required — this repo is public.

## Relationship to Organization-Specific Skill Repos

This repo contains only agnostic skills. Organization-specific skills (internal tooling, proprietary workflows, private context) belong in a private org repo. The `cloud-computer-setup` skill explains how to configure a machine to pull from both sources.
