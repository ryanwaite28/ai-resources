# PROJECT_TEMPLATE_GUIDE.md

> This guide defines the standard structure, intent, and content requirements for `PROJECT.md` and `CLAUDE.md` files used in AI-assisted portfolio projects. It is used by Claude Code (or any AI coding assistant) to scaffold new projects from an idea/prompt.
>
> **Reference projects**: [PixiCred](https://github.com/ryanwaite28/ai-projects-pixicred/blob/main/PROJECT.md) · [Duseum](https://github.com/ryanwaite28/ai-projects-duseum/blob/main/PROJECT.md) · [Qulene](https://github.com/ryanwaite28/ai-projects-qulene/blob/main/PROJECT.md)

---

## Purpose

`PROJECT.md` is the **single source of truth** for a project. Every architectural decision, data model, API contract, infrastructure configuration, business rule, and implementation plan lives here. AI coding assistants must generate specs, implementation tasks, and code directly from this document. Do not store project decisions anywhere else.

`CLAUDE.md` is the **AI session primer** — a concise, opinionated set of rules that tells Claude Code exactly how to behave in the repo: coding standards, what not to do, how to run things, and which patterns are canonical.

---

## PROJECT.md — Section Guide

### Document Header

Start with a title block containing:
- Project name and tagline
- `Project Master Document v1.0`
- A blockquote declaring this as the single source of truth and directing AI tools to generate code from it

---

### Table of Contents

Every `PROJECT.md` uses this numbered structure:

```
1. Project Overview
2. Functional Requirements
3. Non-Functional Requirements
4. System Architecture
5. AWS Infrastructure & Resources
6. Software Design
7. Data Model
8. API Documentation
9. DevOps & Deployment
10. Local Development
11. Implementation Plan
12. Project Rules & AI-IDE Guidelines
13. Infrastructure Cost Estimates
14. Testing Plan
```

---

### Section 1 — Project Overview

**Purpose**: Establish what the project is, how it works end-to-end, and its identity.

**Required subsections**:

- **1.1 What Is [Project]?** — A portfolio note (honest about scope) followed by a 2–3 sentence description of what the system does and what domain it operates in.

- **1.2 How It Works** — A plaintext flow diagram (code block using ASCII arrows `→`) showing the full user journey from entry point to outcome. Cover all major actors and state transitions. Example structure:
  ```
  ACTOR
    → Action step
    → Async/system step
    → Outcome A (happy path)
    → Outcome B (failure/alternate path)
  ```

- **1.3 Key Entities** — A markdown table of the core domain objects with their descriptions. Columns: `Entity | Description`.

- **1.4 Domain-Specific Logic** — Any deterministic/mock rules unique to this domain (e.g. credit scoring rules, mock data strategies). Makes the system testable and demos predictable.

- **1.5 Project Identity** — A table with: Project name, Domain, API URLs (dev + prod), AWS Account, AWS Region, Environments, Resource prefix, Email sender (if applicable).

- **1.6 Mission** — One sentence: what engineering skills this project demonstrates.

---

### Section 2 — Functional Requirements

**Purpose**: Enumerate every feature the system must support, traceable by ID.

**Format**: Requirements are written as bullet lists with coded IDs: `FR-{DOMAIN}-{NUM}`. Example domains: `APP`, `ACC`, `TXN`, `AUTH`, `EMAIL`, `NOTIF`.

**Required subsections** (adapt to your domain):

- One subsection per major feature domain (e.g. User Management, Core Business Flow, Notifications, Authentication, Frontend Application)
- Each subsection title is numbered: `2.1`, `2.2`, etc.
- Each requirement is a bullet: `- **FR-XXX-01**: Description of the requirement.`
- Be concrete and implementation-specific — include field names, status values, formulas, validation rules, idempotency keys, etc.
- Cover the full lifecycle: creation, reads, updates, deletion, edge cases, error conditions
- Email notifications get their own subsection with a numbered template per email type

**Key areas to always cover**:
- Core entities: CRUD, lifecycle state machine, status transitions
- Async operations (if any): what triggers them, what they produce
- Idempotency: how repeated operations are deduplicated
- Authentication & authorization: who can access what
- Frontend pages (if applicable): each route/page as its own FR
- Scheduled/background jobs (if any): triggers, behavior, idempotency

---

### Section 3 — Non-Functional Requirements

**Purpose**: Document cross-cutting constraints that apply system-wide.

**Format**: Bullet list with coded IDs: `NFR-{NUM}`.

**Always include**:
- `NFR-01` — Async processing (if any jobs run out-of-band)
- `NFR-02` — Idempotency policy
- `NFR-03` — Cost efficiency (portfolio = near-zero cost at rest, all Lambda-based)
- `NFR-04` — Testability (service layer framework-agnostic, independently testable)
- `NFR-05` — Local dev parity (emulators for cloud services)
- `NFR-06` — Environment isolation (dev/prod in same AWS account, name-prefixed)
- `NFR-07` — Observability (structured JSON logs, no silent failures)
- `NFR-08` — Security (no secrets in code, Secrets Manager, least-privilege IAM)
- `NFR-09` — **Service Layer Supremacy**: define and enforce the architectural rule that all business logic lives in `src/service/` only

---

### Section 4 — System Architecture

**Purpose**: Describe the system's runtime topology in two forms: ideal production architecture and the cost-optimized portfolio architecture actually deployed. Include a hard callout of the core architectural rule.

**Required subsections**:

- **4.1 Architecture Philosophy** — State the two-tier division (Dispatch Layer vs. Service Layer) and restate Service Layer Supremacy as a callout block:
  ```
  > ⚠ Service Layer Supremacy — Core Architectural Rule
  > [explanation of the rule]
  ```

- **4.2 Production Architecture** — ASCII art diagram of what this would look like fully funded (ECS/ALB, Multi-AZ RDS, WAF, etc.). Include a brief "Why [technology] in production" note.

- **4.3 Portfolio Architecture** — ASCII art diagram of what's actually deployed (Lambdas instead of ECS, single-AZ RDS, etc.). Include a trade-off comparison table:
  | Concern | Production | Portfolio |
  |---------|-----------|-----------|

- **4.4 Async Event Flow** — Full event chain diagrams for every async operation. Format:
  ```
  TRIGGER
    → [Handler] action
        → [Service] operation
            ├─ Path A → outcome
            └─ Path B → outcome
  ```

- **4.5 Local Development Architecture** — Short diagram showing local stack (Express + Docker + queue poller).

---

### Section 5 — AWS Infrastructure & Resources

**Purpose**: Be the IaC source of truth. Every resource that Terraform will create is listed here.

**Required subsections**:

- **5.1 Resource Naming Convention** — Pattern: `{project}-{env}-{descriptor}`. Provide examples.

- **5.2 Standard Tags** — Terraform tag block (Project, Environment, ManagedBy).

- **5.3 AWS Services Used** — Table of every AWS resource: `Service | Resource | Purpose`. Include all Lambdas, SQS queues + DLQs, SNS topics, RDS, SES, EventBridge rules, Secrets Manager, CloudWatch, S3 (state + frontend), CloudFront, ACM, Route 53.

- **5.4 Lambda Configuration** — Table: `Lambda | Memory | Timeout | Trigger`

- **5.5 SQS Queue Configuration** — Table: `Queue | Visibility Timeout | Max Receive Count | DLQ`

- **5.6 RDS Configuration** — Table comparing Portfolio vs. Production settings: instance class, Multi-AZ, storage, backup, engine.

- **5.7 IAM Roles — Least Privilege Summary** — Table: `Lambda | Key Permissions`. One dedicated execution role per Lambda. No shared roles.

---

### Section 6 — Software Design

**Purpose**: Define the full repository structure and internal contracts.

**Required subsections**:

- **6.1 Repository Structure** — Full directory tree as a code block. Include every significant file. Group by: `prisma/`, `src/service/`, `src/db/`, `src/handlers/api/`, `src/handlers/sqs/`, `src/clients/`, `src/emails/`, `src/types/`, `local/`, `frontend/`, `infra/terraform/`, `scripts/`.

- **6.2 Service Layer Internal RPC Contract** — The complete TypeScript `ServiceAction` union type listing every action the service layer can perform. This is the interface contract between dispatch Lambdas and the service layer. Format:
  ```typescript
  type ServiceAction =
    | { action: 'actionName'; payload: InputType }
    | ...
  ```

- **6.3 Tech Stack** — Table: `Layer | Technology`. Cover backend language, ORM, auth, email rendering, AWS SDKs, local API server, build tool, testing, local emulation, IaC, CI/CD, and frontend (framework, routing, state, UI, build, hosting).

---

### Section 7 — Data Model

**Purpose**: Define every database table as production-ready DDL. This is the schema source of truth.

**Format**: One `#### tablename` subsection per table, each containing a `CREATE TABLE` SQL block with all columns, types, constraints, defaults, and index definitions. Include comments on non-obvious columns.

**Always include**:
- Primary keys as `UUID DEFAULT gen_random_uuid()`
- `created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()` on all tables
- Foreign key constraints with `REFERENCES`
- Indexes on all FK columns and any columns used in WHERE clauses
- Unique indexes for idempotency keys
- Inline SQL comments (`-- comment`) for columns whose purpose isn't obvious

---

### Section 8 — API Documentation

**Purpose**: Define every HTTP endpoint as a contract.

**Required subsections**:

- **8.1 Base URLs** — Table: `Environment | API URL | Web App URL` for local, dev, prod.

- **8.2 Response Envelope** — Standard success/error JSON envelope structure.

- **8.3 Endpoints** — Group by domain (Auth, Core Entity A, Core Entity B, Admin/Ops). For each group, a table: `Method | Path | Description`. Mark JWT-required routes with 🔒. Include a note explaining how JWT auth is validated.

- **8.4 Error Codes** — Table: `Code | HTTP Status | Meaning`. Cover all domain-specific errors plus standard ones (UNAUTHORIZED, FORBIDDEN, VALIDATION_ERROR, INTERNAL_ERROR, NOT_FOUND variants).

---

### Section 9 — DevOps & Deployment

**Purpose**: Define how code moves from local to cloud.

**Required subsections**:

- **9.1 Environments** — Table: `Environment | Purpose | Compute`

- **9.2 CI/CD (GitHub Actions)** — Full pipeline as a code block showing the trigger, steps, and any manual approval gates. Include both backend and frontend. Include migration workflow separately.

- **9.3 Terraform Remote State Bootstrap** — Commands to run once before provisioning.

- **9.4 Terraform Module Assessment** — Notes on any external Terraform modules considered and the decision made (use or build custom).

- **9.5 Build** — The key npm scripts: build, test, deploy:dev, deploy:prod.

---

### Section 10 — Local Development

**Purpose**: Enable any developer (or AI agent) to run the full stack locally.

**Required subsections**:

- **10.1 Prerequisites** — Software required: Docker + Compose, Node.js version, AWS CLI.

- **10.2 Starting the Stack** — The exact `docker-compose up` command + seed commands.

- **10.3 docker-compose.yml (summary)** — The full compose file (or a representative summary) as a code block. Include all services: postgres, localstack/ministack, service, api, worker.

- **10.4 Environment Variables (`.env.example`)** — All env vars with example values and inline comments. Note which vars are local-only vs. pulled from Secrets Manager in cloud environments.

- **10.5 Testing the Async Flow Locally** — `curl` examples covering the main user journey end-to-end.

---

### Section 11 — Implementation Plan

**Purpose**: A phased, work-backwards plan that breaks the project into deliverable chunks. Each phase builds on the last.

**Format**: Numbered phases as `### Phase N — Name`. Each phase contains a checklist `- [ ] task`.

**Standard phase sequence**:

- **Phase 0 — Project Scaffold**: repo init, docker-compose, Prisma setup, Terraform bootstrap, `.env.example`, `CLAUDE.md`, `PROJECT.md`, CI skeleton.
- **Phase 1 — Data Model & Service Layer Foundation**: full Prisma schema, typed query functions, service layer skeleton, unit test setup.
- **Phase 2–N — Feature Phases**: one phase per major functional domain (e.g. Auth, Core Entity, Payments, Notifications, Scheduled Jobs).
- **Phase N-1 — API Gateway & Full Wiring**: all Lambda handlers, `service.client.ts`, local servers, Terraform API Gateway.
- **Phase N — DevOps & Hardening**: CI/CD, CloudWatch alarms, Secrets Manager population, post-Terraform setup steps, README.
- **Final Phase — Frontend** (if applicable): depends on API being live. One checklist item per FR-FE-* requirement.

---

### Section 12 — Project Rules & AI-IDE Guidelines

**Purpose**: Explicit rules for any AI coding assistant working in this repo. These prevent hallucinated architecture, wrong file placement, and broken patterns.

**Required rules to always include**:

1. **Service Layer Supremacy** (restate): all business logic in `src/service/`. Lambda handlers do three things only: parse input, call service, return result.
2. **No direct DB access outside `src/db/`**: all DB queries go through typed query functions.
3. **No secrets in code**: use env vars, never hardcode.
4. **TypeScript strict mode**: no `any`, no implicit returns, explicit types on all function signatures.
5. **Error handling**: all errors bubble up as typed error objects or result types; no unhandled promise rejections.
6. **Idempotency**: every write operation that accepts an `idempotencyKey` must enforce uniqueness.
7. **Test coverage**: every service layer function must have a unit test.
8. **Migration discipline**: never edit migration files after they're created; create new migrations instead.
9. **Environment parity**: local behavior must match AWS behavior; use the same code paths.
10. **AI guardrails**: explicitly list what Claude/Cursor/Copilot must NOT do (e.g. no creating new patterns not in this document, no generating Lambda handlers that contain business logic).

---

### Section 13 — Infrastructure Cost Estimates

**Purpose**: Provide honest cost transparency for the portfolio deployment.

**Format**: Two tables — one for Portfolio (actual), one for Production (ideal).

**Cover**: Lambda invocations, RDS instance, SQS/SNS, Secrets Manager, CloudWatch, S3, CloudFront, Route 53. Show monthly cost estimate per service and a total. Add a note on what drives cost at rest vs. under load.

---

### Section 14 — Testing Plan

**Purpose**: Define what gets tested and how.

**Required subsections**:

- **Unit Tests**: Service layer functions tested in isolation with mocked DB and AWS clients. Tool: Vitest.
- **Integration Tests**: Full flows against a real Postgres (Testcontainers). Cover happy path + error paths for each major domain.
- **E2E Tests**: HTTP-level tests against the running local stack. Cover the full user journey.
- **Test Data Strategy**: How synthetic data is generated (seed script, deterministic mock inputs).
- **CI Test Execution**: Which tests run in CI and at which stage.

---

## CLAUDE.md — Structure Guide

`CLAUDE.md` is short, direct, and opinionated. It is the first file Claude Code reads when starting a session. It should answer: "What is this project, how do I run it, and what rules do I follow?"

### Required Sections

**1. Project Summary** (3–5 sentences)
What the project is, what domain it's in, the tech stack in one line, and where the full spec lives.

```markdown
## Project Summary
[Project name] is a [description]. It uses [stack]. See PROJECT.md for the full spec.
```

**2. Quick Start**
The exact commands to get local dev running:
```markdown
## Quick Start
docker-compose up -d
npm install
npm run db:migrate
npm run dev
```

**3. Architecture Rules** (the non-negotiables)
```markdown
## Architecture Rules
- ALL business logic lives in `src/service/`. Lambda handlers only: parse input → call service → return result.
- ALL DB access goes through `src/db/queries/`. No Prisma calls outside this directory.
- NEVER put domain logic, DB queries, or side effects in Lambda handlers.
```

**4. Key Commands**
```markdown
## Key Commands
- `npm run build` — esbuild bundles all Lambdas
- `npm run test` — vitest unit tests
- `npm run test:integration` — integration tests (requires Docker)
- `npm run db:migrate` — run pending migrations
- `npm run db:studio` — open Prisma Studio
- `npm run deploy:dev` — deploy to dev environment
```

**5. File Conventions**
```markdown
## File Conventions
- Service layer: `src/service/{domain}.service.ts`
- DB queries: `src/db/queries/{domain}.queries.ts`
- Lambda handlers (API): `src/handlers/api/{domain}.handler.ts`
- Lambda handlers (SQS): `src/handlers/sqs/{domain}.handler.ts`
- Email templates: `src/emails/templates/{name}.hbs`
- Tests: co-located as `{file}.test.ts` or in `tests/`
```

**6. DO NOT**
```markdown
## DO NOT
- Do not add business logic to Lambda handlers
- Do not call Prisma directly outside `src/db/`
- Do not hardcode secrets, ARNs, or environment-specific values
- Do not create new architectural patterns not defined in PROJECT.md
- Do not modify existing migration files — create new ones
- Do not use `any` type — use explicit TypeScript types
```

**7. Environment Notes**
```markdown
## Environment Notes
- Local: Docker + MiniStack (LocalStack-compatible). AWS calls go to `http://localhost:4566`.
- Dev/Prod: Real AWS. Secrets pulled from Secrets Manager at cold start.
- The service layer is unaware of its runtime target (Lambda vs. Express). Same code, different adapters.
```

---

## Generation Checklist

When scaffolding a new project from an idea, work through these in order:

- [ ] Name the project and define the core domain problem
- [ ] Write Section 1 (Overview) — entities, flow, identity
- [ ] Write Section 2 (Functional Requirements) — enumerate all FRs by domain
- [ ] Write Section 3 (NFRs) — fill in standard NFRs, adjust for domain
- [ ] Write Section 4 (Architecture) — ASCII diagrams for prod + portfolio + async flows
- [ ] Write Section 5 (AWS Resources) — every resource, Lambda config, SQS config, IAM
- [ ] Write Section 6 (Software Design) — repo structure + ServiceAction type + tech stack
- [ ] Write Section 7 (Data Model) — full DDL for every table
- [ ] Write Section 8 (API Docs) — all endpoints, error codes
- [ ] Write Section 9 (DevOps) — CI/CD pipeline, build commands
- [ ] Write Section 10 (Local Dev) — docker-compose, env vars, curl examples
- [ ] Write Section 11 (Implementation Plan) — phased checklist working backwards from done
- [ ] Write Section 12 (Project Rules) — AI guardrails
- [ ] Write Section 13 (Cost Estimates) — portfolio vs. production tables
- [ ] Write Section 14 (Testing Plan) — unit, integration, e2e strategy
- [ ] Write `CLAUDE.md` — summary, quick start, rules, commands, conventions

---

## Quality Bar

A well-written `PROJECT.md` should be comprehensive enough that:

1. A developer who has never seen the project can set up local dev and run the full flow using only `PROJECT.md` and `CLAUDE.md`.
2. An AI coding assistant can generate any Lambda handler, service function, or DB migration without asking clarifying questions.
3. A Terraform engineer can provision the full cloud infrastructure from Section 5 alone.
4. A QA engineer can write integration tests from Section 8 (API docs) alone.

If any of these four conditions aren't met, the document isn't done.