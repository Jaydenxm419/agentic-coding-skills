# Agentic Development Workflow

## Purpose

This workflow is a portable, repeatable blueprint for software development projects of any type. It enforces clean architecture, test-first development, iterative design refinement, and continuous documentation synchronization. Every prompt file is designed to ask you targeted questions that eliminate ambiguity before a single line of code is written. A hierarchical task list (`docs/tasks.md`) is generated during design and actively maintained through implementation as the single source of truth for project progress. Before any implementation work begins on a task, the agent runs a **Human Intervention Pre-Flight Check** that surfaces all non-code setup (account creation, API keys, webhooks, local services, IAM grants) and hard-stops until the developer confirms each item is complete.

## How to Use

Copy this entire directory structure into any new project. The `.claude/commands/` folder contains slash command files. Use whichever editor you prefer — the prompts are tool-agnostic.

### Directory Structure

```
project-root/
├── WORKFLOW.md                          ← You are here
├── .claude/commands/                    ← Slash commands for Claude Code / Cursor
│   ├── 01-init.md
│   ├── 02-requirements.md
│   ├── 03-architecture.md
│   ├── 04-design.md
│   ├── 05-integrations.md              ← External services & env config registry
│   ├── 06-test.md
│   ├── 07-implement.md
│   ├── 08-docs-sync.md
│   ├── 09-audit.md
│   ├── 10-github-init.md              ← Scaffold GitHub milestones, issues, sub-issues
│   ├── 11-github-create-pr.md         ← Open a PR for newly-completed work
│   ├── 12-github-sync.md              ← Periodic full reconciliation
│   └── _standards.md
├── docs/
│   ├── project-init.md                  ← Output of /init
│   ├── requirements.md                  ← Output of /requirements
│   ├── architecture.md                  ← Output of /architecture
│   ├── design.md                        ← Output of /design
│   ├── tasks.md                         ← Output of /design (source of truth)
│   ├── integration-registry.md          ← Output of /integrations
│   └── audits/                          ← Timestamped audit reports land here
```

---

## Recommended Command Order

The workflow follows a linear progression with iterative loops at specific stages.

### Phase 1 — Planning (Hard Gate: No code until ALL five planning stages are complete)

| Order | Command | Purpose | Output | Iterative? |
|-------|---------|---------|--------|------------|
| 1 | `/init` | Project identity, type detection, tooling setup | `docs/project-init.md` | No |
| 2 | `/requirements` | Exhaustive requirements gathering through Q&A | `docs/requirements.md` | Yes — loops until sign-off |
| 3 | `/architecture` | System design, folder structure, component boundaries | `docs/architecture.md` | Yes — loops until sign-off |
| 4 | `/design` | UI/UX flows, component specs, API contracts | `docs/design.md` + `docs/tasks.md` | Yes — loops until sign-off |
| 5 | `/integrations` | External services registry, env variable map, connection catalog | `docs/integration-registry.md` | Yes — loops until sign-off |

**Hard gate**: The `/test` and `/implement` commands will refuse to proceed unless `/requirements`, `/architecture`, `/design`, AND `/integrations` specs exist and have been signed off.

### Phase 2 — Implementation (Test-First Cycle)

| Order | Command | Purpose | Iterative? |
|-------|---------|---------|------------|
| 6 | `/test` | Generate test suites from the spec for a target milestone or task | Yes — per task |
| 7 | `/implement` | Run Human Intervention Pre-Flight Check, then write code that passes the tests, then check off subtasks in `docs/tasks.md` | Yes — per task |

**Cycle**: For every task: run `/test` first to create the test suite from the spec, then `/implement` to write code that passes those tests. `/implement` begins with a **Human Intervention Pre-Flight Check** that scans the Integration Registry, the test files, and the architecture/design specs to surface every non-code prerequisite (account creation, API key generation, webhook configuration, local services, IAM permissions, etc.). The checklist is persisted to `docs/tasks.md` under the target task as a `Human Setup:` field. The agent hard-stops and refuses to generate any code until the developer confirms every item is `[x]` in chat. Once cleared, the agent checks off subtasks in `docs/tasks.md` as tests pass, linking each to the implementation file. Repeat per task until the milestone is complete.

### Phase 3 — Maintenance & Sync (Ongoing)

| Order | Command | Purpose | Iterative? |
|-------|---------|---------|------------|
| 8 | `/docs-sync` | Detect and fix documentation drift, update `docs/tasks.md` if scope changed | Yes — after changes |
| 9 | `/audit` | Deep codebase health check against full spec and integration registry | Yes — periodic |
| 10 | `/github-init` | Scaffold GitHub milestones, issues, and sub-issues from `docs/tasks.md`. Static-only — no git operations, works from any branch. | Yes — idempotent |
| 11 | `/github-create-pr` | Detect newly-completed subtasks and open one PR per parent task. Strict-branch-confined; refuses on default branch. | Yes — per task completion |
| 12 | `/github-sync` | Long-running full reconciliation between `docs/tasks.md` and GitHub. Delegates to `/github-init` for missing items and `/github-create-pr` for missing PRs. | Yes — periodic full sweep |

**The three GitHub commands have distinct, focused responsibilities**:

- **`/github-init`** is for scaffolding. It creates and updates milestones, issues, and sub-issues. It performs zero git operations, so it works from any branch. Run it on day one of a project right after `/design` sign-off, or any time `/docs-sync` adds new items. If no remote is configured, it walks the developer through creating one.

- **`/github-create-pr`** is for publishing completed work. It detects newly-completed subtasks (`[x]` with `Impl:` references but no `PR:` field), groups them by parent task, and opens one PR per task. It commits and pushes only on the current branch and refuses to commit when on the default branch. PRs require human review before merging — the skill never enables auto-merge. If issues don't exist for the relevant tasks, this skill auto-invokes `/github-init`'s logic to create them first.

- **`/github-sync`** is the periodic full reconciliation. It walks the entire `docs/tasks.md` hierarchy, identifies every gap, and delegates to `/github-init` (for missing GitHub items) or `/github-create-pr` (for missing PRs). Use it for end-of-sprint cleanup, pre-release verification, or after `/docs-sync` adjustments.

**Composability**: You can run any of the three independently, or chain them. `/github-create-pr` can run standalone — it'll auto-create issues if they don't exist. `/github-sync` orchestrates both `/github-init` and `/github-create-pr` under a unified reconciliation flow.

### Iterative Flow Summary

```
[init] → [requirements ⟲] → [architecture ⟲] → [design ⟲] → [integrations ⟲]
                                                    │                  │
                                             generates tasks.md   HARD GATE
                                                    │                  │
                                                    └──► [github-init] ◄┐
                                                                       │ │
                                                             ┌─── [test ⟲] ◄──┐
                                                             │         │        │
                                                             │   [implement ⟲] ─┘
                                                             │     │
                                                             │     ├── Human Intervention
                                                             │     │   Pre-Flight Check
                                                             │     │   (HARD STOP until
                                                             │     │   every [x] confirmed)
                                                             │     │
                                                             │     ├── Subtask implementation
                                                             │     │
                                                             │     └── Checks off tasks.md
                                                             │   tests pass?
                                                             │     yes │
                                                             │      [github-create-pr] (opens PR)
                                                             │         │
                                                             └──► [docs-sync]
                                                                       │
                                                                   [audit ⟲]
                                                                       │
                                                                 [github-sync ⟲]
                                                                 (full reconciliation)
```

**⟲ = iterative** — the agent will continue asking questions and refining until you explicitly sign off or all checklist items are satisfied.

---

## The Human Intervention Pre-Flight Check

Before `/implement` writes any code for a target task, it runs a Human Intervention Pre-Flight Check. This is a hard gate with no overrides.

### What it does

The agent scans three sources for non-code prerequisites:

1. **Integration Registry** (`docs/integration-registry.md`) — for any subtask with an `Integration:` field, it pulls account requirements, API keys, webhook setup, OAuth registration, redirect URIs, domain verification, and plan/quota requirements.
2. **Test files** (referenced by the target task's subtasks) — it scans for env var assertions, mock fixture requirements, local service dependencies, test database setup, and external test accounts (sandbox modes).
3. **Architecture & design specs** — it scans for infrastructure prerequisites, deployment-target dependencies, auth-flow prerequisites, cross-cutting concerns (logging/monitoring project IDs), and design-driven third-party widget setup.

### How it persists

The resulting checklist is written to `docs/tasks.md` under the target task as a `Human Setup:` field. Each item is concrete and action-oriented, tagged with its source. Example:

```markdown
### T1.2 — OAuth Provider Integration
- Status: NOT STARTED
- Requirement refs: REQ-3, REQ-7
- GitHub: issue #9
- Human Setup:
  - [ ] Create a Google Cloud project and enable the OAuth 2.0 client at https://console.cloud.google.com (Source: Integration Registry — Google OAuth)
  - [ ] Add `http://localhost:3000/auth/callback` to the OAuth client's authorized redirect URIs (Source: Integration Registry — Google OAuth)
  - [ ] Copy `GOOGLE_OAUTH_CLIENT_ID` and `GOOGLE_OAUTH_CLIENT_SECRET` into `.env` (Source: Tests — tests/auth/oauth.test.ts)
  - [ ] Start the local Postgres instance on port 5432 (Source: Architecture — Section 2: Data Architecture)
```

### How it clears

The developer confirms each item is complete in chat. The agent marks each `[x]` with a confirmation timestamp inline. Once every item is `[x]`, code generation begins. The checklist is preserved on the completed task as a historical record.

### What if the developer disputes an item

The agent walks through the source that triggered the item. If genuinely spurious, the item is removed with a brief note. If genuinely required, the agent holds the line — there are no overrides on this gate.

### What if a missed item surfaces mid-implementation

If a test fails because of an environmental issue (e.g., a service isn't running despite being marked `[x]`), the agent stops, adds the item back to the checklist as `[ ]`, and requests reconfirmation before resuming.

---

## The Task Tracker — `docs/tasks.md`

The task tracker is the single source of truth for what needs to be built and what has been completed. It uses a strict three-level hierarchy:

```
Milestone (M1, M2, ...)
  └── Task (T1.1, T1.2, ...)
        └── Subtask (S1.1.1, S1.1.2, ...)
```

### Rules

1. **Generated after `/design`**: The task list is derived from the combined requirements, architecture, and design specs. Every requirement must trace to at least one subtask.
2. **Strict rollup**: A task is only `[x]` when ALL its subtasks are `[x]`. A milestone is only `[x]` when ALL its tasks are `[x]`. No partial credit.
3. **Implementation references**: When a subtask is completed, the agent adds the file path where the implementation lives (e.g., `src/auth/login.ts`).
4. **Test references**: Each subtask links to its corresponding test file.
5. **Human Setup field**: When `/implement` targets a task, it writes a `Human Setup:` field on that task containing every non-code prerequisite. The field persists after task completion as a historical record. See "The Human Intervention Pre-Flight Check" above.
6. **GitHub references**: When `/github-init` runs (or is invoked by `/github-create-pr` or `/github-sync`), each milestone, task, and subtask gets a `GitHub:` field linking to its issue or milestone number.
7. **PR references**: When `/github-create-pr` (or `/github-sync` invoking it) detects newly-completed subtasks, it groups them under the parent task and opens one PR per task. The `PR:` field is added inline for the parent task and each subtask included in that PR.
8. **The `/implement` command reads `docs/tasks.md` before writing any code** to know what to build and to verify all prior subtasks in the current task are complete, AND to verify every `Human Setup:` item on the target task is `[x]`.
9. **The `/implement` command writes to `docs/tasks.md` in two places**: before coding (the `Human Setup:` checklist on the target task) and after passing tests (`[x]`, `Impl:` paths, and status rollups).
10. **Iterative updates**: If `/design` is re-run to refine the spec, `docs/tasks.md` is regenerated or updated to reflect changes. New items are appended; completed items are preserved.
11. **Integration awareness**: Subtasks that involve external services reference the Integration Registry. The `/implement` command consults both `docs/tasks.md` and `docs/integration-registry.md` when working on these subtasks.

### Example Structure

```markdown
## M1 — User Authentication
- Status: IN PROGRESS
- GitHub: milestone #1

### T1.1 — Login Flow
- Status: COMPLETE
- GitHub: issue #5
- PR: #42
- Human Setup:
  - [x] Provision the Postgres test database on localhost:5432 — confirmed 2026-05-10 14:22 (Source: Architecture — Section 2)
  - [x] Add JWT_SIGNING_SECRET to .env — confirmed 2026-05-10 14:25 (Source: Tests — tests/auth/token.test.ts)

- [x] S1.1.1 — Email/password validation logic
  - Requirement: Users must log in with email and password
  - Test: `tests/auth/login-validation.test.ts`
  - Impl: `src/auth/validators.ts`
  - GitHub: issue #6
  - PR: #42
- [x] S1.1.2 — JWT token generation
  - Requirement: System must issue JWT on successful login
  - Test: `tests/auth/token.test.ts`
  - Impl: `src/auth/token-service.ts`
  - GitHub: issue #7
  - PR: #42
- [x] S1.1.3 — Login API endpoint
  - Requirement: POST /api/auth/login returns token
  - Test: `tests/auth/login-endpoint.test.ts`
  - Impl: `src/api/routes/auth.ts`
  - GitHub: issue #8
  - PR: #42

### T1.2 — OAuth Provider Integration
- Status: NOT STARTED
- Integration: Google OAuth (see integration-registry.md)
- GitHub: issue #9
- Human Setup:
  - [ ] Create Google Cloud project and OAuth 2.0 client (Source: Integration Registry — Google OAuth)
  - [ ] Add redirect URIs to OAuth client (Source: Integration Registry — Google OAuth)
  - [ ] Copy GOOGLE_OAUTH_CLIENT_ID and GOOGLE_OAUTH_CLIENT_SECRET into .env (Source: Tests — tests/auth/oauth.test.ts)

- [ ] S1.2.1 — OAuth redirect flow
  - Requirement: Users can log in with Google
  - Test: (pending /test)
  - Impl: (pending /implement)
  - GitHub: issue #10
- [ ] S1.2.2 — OAuth callback handler
  - Requirement: System processes OAuth callback and creates session
  - Test: (pending /test)
  - Impl: (pending /implement)
  - GitHub: issue #11
```

---

## Standards Enforcement

Every command references `_standards.md`, which contains:
- Code quality checklist applicable to all project types
- Naming conventions
- Error handling requirements
- Documentation requirements
- Clean architecture enforcement rules
- Integration registry enforcement rules (including the Human Setup hard gate on `/implement`)
- Task tracker standards and status rules (including the `Human Setup:` field)
- CI/CD considerations for team workflows

---

## Key Principles

1. **No ambiguity reaches code.** Every question the agent asks you is designed to resolve a decision that would otherwise become a bug or a refactor.
2. **Test-first, always.** Tests are written from the spec, not from the implementation. Code is written to satisfy tests.
3. **Configuration before code.** Every external service, API key, database connection, and environment variable is documented in the Integration Registry before any implementation begins. The agent never invents configuration on the fly.
4. **Human setup before code.** Every task's non-code prerequisites are surfaced, persisted, and confirmed before `/implement` writes a single line. The agent cannot create accounts, generate keys, or grant permissions for you — and it won't pretend it can. The pre-flight check is a hard stop with no overrides.
5. **Task list is the source of truth.** The agent reads `docs/tasks.md` before writing code, checks off subtasks as tests pass, and links every completed subtask to its implementation file.
6. **Documentation is a living artifact.** The `/docs-sync` command timestamps and audits every change. Spec drift is caught, not ignored.
7. **Clean architecture is non-negotiable.** The architecture phase produces the folder structure, and that structure enforces separation of concerns regardless of project type.
8. **Commands are independent but aware.** You can run any command at any time, but the agent will warn you if prerequisite artifacts are missing and recommend the correct order.
