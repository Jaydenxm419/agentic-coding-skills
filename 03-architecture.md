# /architecture — System Architecture

> **Purpose**: Design the complete system architecture including component boundaries, data models, API contracts, folder structure, and cross-cutting concerns. The architecture spec, combined with the design spec, must contain enough detail that implementation requires zero guesswork.
> **Prerequisites**: `docs/requirements.md` must exist and be signed off. If missing or unsigned, warn and recommend running `/requirements` first.
> **Output**: `docs/architecture.md`
> **Iterative**: Yes — loops through architecture decisions until the developer signs off.

---

## Instructions for the Agent

You are designing the system architecture. Read `docs/project-init.md` and `docs/requirements.md` before starting.

**Behavior rules:**
1. Present architecture decisions one section at a time. Get confirmation before moving to the next.
2. Every decision must trace back to a requirement. If a decision has no requirement driving it, flag it as a potential over-engineering concern.
3. Enforce clean architecture principles as defined in `_standards.md`.
4. The developer must sign off on the complete architecture before proceeding.
5. The folder structure defined here becomes the canonical reference — implementation must match it exactly.

---

## Section 1 — High-Level Architecture

Ask the developer:

1. **What architectural pattern fits this project?** Present options based on the project type detected in `/init`:
   - Monolith (layered)
   - Modular monolith
   - Microservices
   - Serverless
   - Event-driven
   - Client-server
   - Other (describe)

2. **What are the major system components?** Draw the boundaries between them.
3. **How do components communicate?** (REST, GraphQL, message queue, direct import, etc.)

Present a high-level component diagram (text-based) for confirmation.

---

## Section 2 — Data Architecture

For each entity identified in requirements:

1. **Define the complete schema** (fields, types, constraints, defaults)
2. **Define relationships** (one-to-one, one-to-many, many-to-many, with join entities if needed)
3. **Define the storage strategy** (database type, table vs. document, indexing strategy)
4. **Define access patterns** (which components read/write which entities, with what frequency)

---

## Section 3 — API Contracts

For every external or internal API boundary:

1. **Endpoint / function signature**: Full path, method, parameters
2. **Request payload**: Schema with types and validation rules
3. **Response payload**: Schema with types, including error responses
4. **Authentication / authorization**: What credentials are required, what permissions are checked
5. **Rate limiting / throttling**: If applicable

---

## Section 4 — Folder Structure

Define the canonical folder structure. This must follow clean architecture as defined in `_standards.md`.

Ask the developer:
> "Based on the project type ([type]) and stack ([stack]), here is the proposed folder structure. Does this match your expectations, or do you want to adjust?"

The folder structure must clearly separate: Domain / core logic, Application / use cases, Infrastructure / external integrations, Presentation / UI / API routes, Shared / utilities.

---

## Section 5 — Cross-Cutting Concerns

Address each:

1. **Error handling strategy**: Error types, propagation rules, user-facing error format
2. **Authentication & authorization**: Method (JWT, session, OAuth), where checks happen
3. **Logging & observability**: What is logged, log levels, monitoring approach
4. **Configuration management**: Environment variables, config files, secrets handling
5. **CI/CD pipeline**: Build steps, test stages, deployment strategy

---

## Section 6 — Architecture Decision Records

For every significant choice made during this phase, document:

| Decision | Options Considered | Choice | Rationale | Requirement Ref |
|----------|--------------------|--------|-----------|-----------------|
| [decision] | [options] | [choice] | [why] | [REQ-xxx] |

---

## Phase: Review & Sign-Off

Present the complete architecture document and the sign-off checklist from `_standards.md`.

When the developer signs off, append:

```markdown
## Sign-Off
- Signed off by: [developer name or "Developer"]
- Date: [timestamp]
- Open items: [none, or list]
```

---

## Post-Sign-Off

Tell the developer:
1. **File created/updated**: `docs/architecture.md`
2. **Next command**: `/design` — to create the UI/UX and component design specification, and generate the project task list
3. **Reminder**: The `/design` phase will produce both `docs/design.md` and `docs/tasks.md` (the hierarchical task tracker). After that, `/integrations` is the final planning step before the hard gate lifts.
