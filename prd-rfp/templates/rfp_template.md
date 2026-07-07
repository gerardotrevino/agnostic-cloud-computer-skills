# Request for Proposal / Technical Specification
**Project:** [Project Name]
**Version:** 1.0
**Date:** [Date]
**Author:** [Author]
**Status:** Draft | Review | Approved
**PRD Reference:** `docs/PRD.md`

---

## 1. Architecture Overview

**System Type:** [Monolith / Microservices / Serverless / Hybrid]

**Component Diagram (text description):**
```
[Frontend] → [API Gateway / BFF] → [Backend Services]
                                        ↓
                                   [Database]
                                        ↓
                                  [External APIs]
```

**Components:**
| Component | Responsibility | Technology |
|-----------|---------------|------------|
| Frontend | [What it does] | [Framework] |
| Backend API | [What it does] | [Framework] |
| Database | [What it stores] | [DB engine] |
| Cache | [What it caches] | [Redis / etc] |
| Queue | [What it queues] | [if applicable] |

---

## 2. Tech Stack

| Layer | Technology | Version | Rationale |
|-------|------------|---------|-----------|
| Frontend | [e.g., React + TypeScript] | [version] | [Why this choice] |
| Backend | [e.g., Node.js + Fastify] | [version] | [Why this choice] |
| Database | [e.g., PostgreSQL] | [version] | [Why this choice] |
| Cache | [e.g., Redis] | [version] | [Why this choice] |
| Auth | [e.g., JWT + bcrypt] | [version] | [Why this choice] |
| Hosting | [e.g., Cloud Computer / Railway] | - | [Why this choice] |
| CI/CD | [e.g., GitHub Actions] | - | [Why this choice] |
| Containerization | [e.g., Docker + Compose] | [version] | [Why this choice] |

---

## 3. Data Models

### Entity: [EntityName]
```typescript
interface EntityName {
  id: string;           // UUIDv7, primary key
  createdAt: Date;      // auto-set on insert
  updatedAt: Date;      // auto-set on update
  deletedAt?: Date;     // soft delete
  // ... fields
}
```

### Entity: [EntityName2]
```typescript
interface EntityName2 {
  id: string;
  // ...
}
```

**Relationships:**
- `EntityName` has many `EntityName2` (via `entityNameId` FK)

---

## 4. API Contracts

### Authentication
All protected endpoints require: `Authorization: Bearer <token>`

### Endpoints

#### POST /api/v1/[resource]
**Description:** [What this does]
**Auth:** Required / Public

**Request:**
```json
{
  "field": "type"
}
```

**Response 200:**
```json
{
  "id": "uuid",
  "field": "value"
}
```

**Errors:**
| Code | Meaning |
|------|---------|
| 400 | Validation error |
| 401 | Unauthorized |
| 404 | Not found |

#### GET /api/v1/[resource]/:id
[Repeat pattern for each endpoint]

---

## 5. Auth & Security

**Authentication Strategy:** [JWT / Session / OAuth2 / API Key]
**Token Expiry:** [e.g., Access: 15min, Refresh: 7 days]
**Password Hashing:** [e.g., bcrypt, cost factor 12]

**Authorization Model:**
| Role | Permissions |
|------|-------------|
| [Role 1] | [What they can do] |
| [Role 2] | [What they can do] |

**Data Protection:**
- Encryption at rest: [Yes/No — method]
- Encryption in transit: [TLS 1.3]
- PII handling: [How PII is stored/masked]
- Secrets management: [.env / Vault / etc]

---

## 6. Infrastructure

**Environments:**
| Environment | URL | Purpose |
|-------------|-----|---------|
| Development | `localhost:[port]` | Local dev |
| Staging | `http://[ip]:[port]` | QA / testing |
| Production | `http://[ip]:[port]` | Live |

**Deployment:**
- Containerized with Docker Compose
- Port allocation: [Block from PORT_REGISTRY.md]
- Auto-restart: `unless-stopped`

**Scaling Strategy:** [Vertical / Horizontal / N/A for MVP]

---

## 7. Integrations

| Service | Purpose | Auth Method | Docs |
|---------|---------|-------------|------|
| [Service 1] | [What it does] | [API Key / OAuth] | [URL] |
| [Service 2] | [What it does] | [API Key / OAuth] | [URL] |

---

## 8. Implementation Plan

### Phase 1: Foundation (Week 1–2)
- [ ] Project scaffold and Docker setup
- [ ] Database schema and migrations
- [ ] Auth system (register, login, JWT)
- [ ] Core API skeleton

### Phase 2: Core Features (Week 3–5)
- [ ] [Feature 1 from PRD FR-001]
- [ ] [Feature 2 from PRD FR-002]
- [ ] [Feature 3 from PRD FR-003]

### Phase 3: Polish & Launch (Week 6)
- [ ] Frontend integration
- [ ] E2E tests
- [ ] Performance testing
- [ ] Deployment to production

---

## 9. Risks & Mitigations

| # | Risk | Probability | Impact | Mitigation |
|---|------|-------------|--------|------------|
| 1 | [Risk description] | High/Med/Low | High/Med/Low | [How to mitigate] |
| 2 | [Risk description] | High/Med/Low | High/Med/Low | [How to mitigate] |

---

## 10. Acceptance Criteria

### Phase 1 Done When:
- [ ] All unit tests pass
- [ ] Auth endpoints return correct status codes
- [ ] Database migrations run cleanly

### Phase 2 Done When:
- [ ] All functional requirements FR-001 through FR-00X are implemented
- [ ] Integration tests pass
- [ ] No P0/P1 bugs open

### Launch Done When:
- [ ] E2E tests pass on staging
- [ ] Performance targets met (see NFRs)
- [ ] Security review completed
- [ ] Documentation updated
