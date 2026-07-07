---
name: prd-rfp
description: >-
  Structured workflow for creating Product Requirements Documents (PRD) and
  Request for Proposal / Technical Spec (RFP) documents for software projects.
  Use when starting a new project, planning a feature set, or preparing technical
  specifications that will drive development. Triggers on: write a PRD, create
  requirements, plan the project, define the spec, RFP, product requirements,
  technical specification.
---

# PRD / RFP Skill

This skill guides the creation of two complementary planning documents that together drive a software project from idea to implementation:

- **PRD (Product Requirements Document):** Defines *what* the product does — user stories, features, acceptance criteria, and non-functional requirements.
- **RFP (Request for Proposal / Technical Spec):** Defines *how* the product is built — architecture, tech stack, data models, API contracts, and implementation plan.

---

## When to Use

Use this skill at the **start of any new project or major feature** before writing a single line of code. The output documents become the contract between product and engineering, and the input for the `writing-plans` skill.

---

## Workflow

### Stage 1: Discovery (Gather Context)

Ask the user the following questions **one at a time** (never all at once):

1. **What problem does this product solve?** Who has the problem, and why does it matter?
2. **Who are the primary users?** Describe each user persona (role, goals, pain points).
3. **What are the core features?** List the top 5–10 things the product must do.
4. **What are the constraints?** (Timeline, budget, team size, compliance, integrations)
5. **What does success look like?** What metrics or outcomes define a successful launch?
6. **What is the tech stack preference?** (Languages, frameworks, databases, hosting)
7. **Are there existing systems to integrate with?** (APIs, databases, auth providers)

After gathering answers, summarize back to the user and ask for confirmation before proceeding.

---

### Stage 2: Write the PRD

Use the template at `templates/prd_template.md`. Fill every section — no placeholders.

**PRD Sections:**
1. Executive Summary — one paragraph, the elevator pitch
2. Problem Statement — the pain being solved, with evidence
3. Goals & Success Metrics — measurable outcomes (OKRs or KPIs)
4. User Personas — name, role, goals, frustrations for each type
5. User Stories — "As a [persona], I want to [action] so that [outcome]" for every feature
6. Functional Requirements — numbered list of what the system must do
7. Non-Functional Requirements — performance, security, scalability, compliance
8. Out of Scope — explicit list of what is NOT being built
9. Open Questions — unresolved decisions that need answers before dev starts

Save to: `docs/PRD.md` in the project repo.

---

### Stage 3: Write the RFP / Technical Spec

Use the template at `templates/rfp_template.md`. Fill every section — no placeholders.

**RFP / Tech Spec Sections:**
1. Architecture Overview — diagram description + component list
2. Tech Stack — language, framework, database, hosting, CI/CD (with rationale)
3. Data Models — entity definitions with fields, types, and relationships
4. API Contracts — endpoint list with method, path, request/response shapes
5. Auth & Security — authentication strategy, authorization model, data protection
6. Infrastructure — deployment topology, environments (dev/staging/prod), scaling
7. Integrations — external APIs, third-party services, webhooks
8. Implementation Plan — phased milestones with deliverables
9. Risks & Mitigations — top 5 risks with mitigation strategies
10. Acceptance Criteria — definition of done for each milestone

Save to: `docs/RFP.md` in the project repo.

---

### Stage 4: Review Gate

After writing both documents, run a self-review checklist:
- [ ] No "TBD" or "TODO" in any section
- [ ] Every user story has a corresponding functional requirement
- [ ] Every functional requirement maps to a data model or API endpoint
- [ ] Tech stack choices are justified
- [ ] All external integrations are listed with their auth method
- [ ] Success metrics are measurable (not vague)

Present documents to the user for approval:
> "PRD and RFP written and saved to `docs/`. Please review both documents and confirm they accurately capture the project. Once approved, I'll use the `writing-plans` skill to create the detailed implementation plan."

Wait for user approval. Iterate on feedback.

---

### Stage 5: Handoff to writing-plans

Once approved, invoke the `writing-plans` skill with the RFP as the spec input. The PRD provides the "why", the RFP provides the "how", and writing-plans produces the task-by-task implementation roadmap.

---

## Key Principles

- **One question at a time** — never overwhelm the user with a wall of questions.
- **No placeholders** — every section must be complete before presenting to the user.
- **PRD drives RFP** — every technical decision in the RFP must trace back to a requirement in the PRD.
- **Explicit out-of-scope** — what you don't build is as important as what you do.
- **Measurable success** — every goal must have a metric attached.

---

## Reference Files

- `templates/prd_template.md` — Full PRD template with section prompts
- `templates/rfp_template.md` — Full RFP/Tech Spec template with section prompts
- `references/prd_examples.md` — Example PRD sections for common project types
