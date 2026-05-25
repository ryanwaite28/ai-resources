# Skill: Project Scaffolding (PROJECT.md + CLAUDE.md)

## Purpose

This skill directs a Claude Code session to generate a complete `PROJECT.md` and `CLAUDE.md` for a new project from a given idea or prompt. The output mirrors the structure and depth of the reference projects:

- [PixiCred](https://github.com/ryanwaite28/ai-projects-pixicred/blob/main/PROJECT.md)
- [Duseum](https://github.com/ryanwaite28/ai-projects-duseum/blob/main/PROJECT.md)
- [Qulene](https://github.com/ryanwaite28/ai-projects-qulene/blob/main/PROJECT.md)

The full structural spec lives in `PROJECT_TEMPLATE_GUIDE.md` (co-located with this skill).

---

## When to Use This Skill

Use when:
- Starting a new portfolio/AI-assisted project from scratch
- Given a project idea or description and need to generate planning documents
- Onboarding a new project into the `ai-projects-*` pattern

Triggers: phrases like "scaffold a new project", "generate PROJECT.md for", "create project docs for", "set up a new AI project".

---

## Inputs

Claude Code should collect the following before generating. If any are missing from the prompt, ask for them before proceeding:

| Input | Required | Notes |
|-------|----------|-------|
| `project_name` | Yes | Short CamelCase or PascalCase name (e.g. "PixiCred", "Duseum") |
| `project_idea` | Yes | 1–3 sentence description of what the project does |
| `domain` | Yes | The problem domain (e.g. fintech, ecommerce, SaaS, social, logistics) |
| `tech_stack_overrides` | No | Any deviations from the default stack (default: TypeScript, Node 20, Prisma, Angular, AWS serverless) |
| `aws_account_id` | No | If known; otherwise use placeholder `000000000000` |
| `domain_name` | No | Custom domain (e.g. `myproject.com`); otherwise derive from project name |

---

## Instructions

### Step 1 — Read the guide

Before generating anything, read `PROJECT_TEMPLATE_GUIDE.md` in full. This file defines the exact structure, subsection requirements, format conventions, and quality bar for both `PROJECT.md` and `CLAUDE.md`.

### Step 2 — Analyze the project idea

Before writing, reason through the following and keep notes internally:

1. **Core entities**: What are the 3–6 main domain objects? What are their lifecycles?
2. **User flows**: What does each actor do, step by step? What's async?
3. **Functional domains**: What are the natural groupings of features (e.g. Auth, Orders, Notifications)?
4. **Async operations**: Are there background jobs, scheduled tasks, or event-driven flows?
5. **Data relationships**: Which entities reference which? What needs idempotency keys?
6. **API shape**: What are the main REST endpoints? Which require auth?
7. **Infrastructure**: What AWS services are needed? How does local emulation work?

### Step 3 — Generate PROJECT.md

Write all 14 sections in order. Follow these rules strictly:

**Section 1 — Project Overview**
- Include the portfolio note (honest about scope and intent)
- Write the How It Works flow using `→` arrows in a code block
- Table of key entities with descriptions
- Any domain-specific mock/deterministic logic (makes demos testable)
- Project Identity table (use placeholder values if specifics aren't known)
- One-sentence Mission

**Section 2 — Functional Requirements**
- Minimum 6 subsections for non-trivial projects
- Each FR has a unique coded ID: `FR-{DOMAIN}-{NUM}`
- Be implementation-specific: include field names, status enums, formulas, error codes
- Cover the full lifecycle of each entity including edge cases
- Include a frontend subsection (FR-FE-*) if a UI is part of scope
- Include an email/notification subsection if the project sends notifications

**Section 3 — Non-Functional Requirements**
- Use the standard 9 NFRs from the guide
- Adapt NFR-09 (Service Layer Supremacy) to this project's domain language

**Section 4 — System Architecture**
- Both diagrams (production and portfolio) using ASCII art
- Service Layer Supremacy callout block is mandatory
- Full async event flow chains for every async operation
- Local development architecture diagram

**Section 5 — AWS Infrastructure**
- Every Lambda, queue, topic, and supporting resource listed in the table
- One DLQ per SQS queue
- IAM roles: one per Lambda, least-privilege, no sharing

**Section 6 — Software Design**
- Full directory tree — no placeholder `...` ellipsis; list actual files
- Complete `ServiceAction` union type for all service operations
- Tech stack table with specific library versions where relevant

**Section 7 — Data Model**
- Full DDL `CREATE TABLE` statements for every entity
- All indexes defined explicitly
- UUID PKs with `gen_random_uuid()`
- Idempotency keys as UNIQUE indexes

**Section 8 — API Documentation**
- All endpoints grouped and tabled
- 🔒 marker on JWT-required routes
- All error codes with HTTP status and meaning

**Section 9 — DevOps & Deployment**
- Full GitHub Actions pipeline as a code block
- Separate migration workflow
- Build command summary

**Section 10 — Local Development**
- Full `docker-compose.yml` in a code block
- Complete `.env.example` with all variables and comments
- `curl` examples covering the main user journey

**Section 11 — Implementation Plan**
- Work backwards: define the end state, then determine the phases needed to reach it
- Phase 0 is always project scaffold
- Last phase (before optional frontend) is always DevOps & hardening
- Frontend is always the final phase if applicable
- Each phase has a checkbox checklist

**Section 12 — Project Rules**
- At minimum: the 10 standard rules from the guide
- Add domain-specific rules where appropriate

**Section 13 — Infrastructure Cost Estimates**
- Two tables: portfolio vs. production
- Per-service monthly cost estimate
- Note on cost at rest vs. under load

**Section 14 — Testing Plan**
- Unit, integration, and e2e coverage strategy
- Tool: Vitest + Testcontainers
- Test data/seed strategy

### Step 4 — Generate CLAUDE.md

After `PROJECT.md` is complete, generate `CLAUDE.md` using the structure defined in the guide:

1. **Project Summary** — 3–5 sentences
2. **Quick Start** — exact commands
3. **Architecture Rules** — the non-negotiables (especially Service Layer Supremacy)
4. **Key Commands** — `npm run *` scripts
5. **File Conventions** — where each type of file lives
6. **DO NOT** — explicit prohibitions for AI assistants
7. **Environment Notes** — local vs. cloud differences

`CLAUDE.md` should be concise (under 150 lines). It is read at the start of every session; it cannot be verbose.

---

## Output

Produce two files:

```
PROJECT.md    — full project specification (~1000–1500 lines for a moderately complex project)
CLAUDE.md     — AI session primer (~80–150 lines)
```

Write both files to the project root. Do not create any other files during this step.

---

## Quality Checks

Before finishing, verify each of these passes:

- [ ] `PROJECT.md` has all 14 sections with the correct numbering
- [ ] All functional requirements have coded IDs (`FR-*`)
- [ ] All non-functional requirements have coded IDs (`NFR-*`)
- [ ] ASCII architecture diagrams are present for production, portfolio, and async flows
- [ ] Every AWS resource is named using the `{project}-{env}-{descriptor}` convention
- [ ] The `ServiceAction` union type covers every service operation referenced in the FRs
- [ ] Every table in the data model has a PK, `created_at`, and all necessary indexes
- [ ] Every API endpoint is listed with method, path, and description
- [ ] The implementation plan phases are ordered correctly (scaffold → data → features → wiring → devops → frontend)
- [ ] `CLAUDE.md` is under 150 lines
- [ ] `CLAUDE.md` includes a DO NOT section
- [ ] Both files can be committed to a new repo immediately

---

## Default Stack Assumptions

Unless overridden by the user, assume:

| Layer | Default |
|-------|---------|
| Backend language | TypeScript (Node.js 20) |
| ORM | Prisma |
| Database | PostgreSQL 15 on RDS |
| API | AWS API Gateway v2 + Lambda |
| Service layer runtime | Private Lambda (portfolio) / ECS Fargate (production) |
| Queue | AWS SQS |
| Events | AWS SNS |
| Email | AWS SES with Handlebars templates |
| Auth | JWT (HS256, 24h expiry) + bcrypt passwords |
| Local emulation | MiniStack (LocalStack-compatible) |
| Frontend framework | Angular 17+ (standalone components) |
| Frontend styling | Tailwind CSS |
| Frontend hosting | S3 + CloudFront |
| IaC | Terraform |
| CI/CD | GitHub Actions |
| Testing | Vitest + Testcontainers |
| Build | esbuild (per Lambda bundle) |

---

## Notes for Claude Code

- Do not ask for approval between sections — generate the full `PROJECT.md` in one pass, then generate `CLAUDE.md`.
- If any input is ambiguous, make a reasonable assumption and note it in the relevant section rather than stopping to ask.
- The reference projects are production-grade in design intent, not tutorial-grade. Match that standard.
- A thin or incomplete document is worse than a long one. Err on the side of more detail.
- Do not truncate sections. If a section needs 30 functional requirements to be complete, write 30.