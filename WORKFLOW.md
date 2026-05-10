# Agentic Development Workflow

## Purpose

This workflow is a portable, repeatable blueprint for software development projects of any type. It enforces clean architecture, test-first development, iterative design refinement, and continuous documentation synchronization. Every prompt file is designed to ask you targeted questions that eliminate ambiguity before a single line of code is written. A hierarchical task list (`docs/tasks.md`) is generated during design and actively maintained through implementation as the single source of truth for project progress.

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
| 7 | `/implement` | Write code that passes the tests, check off subtasks in `docs/tasks.md` | Yes — per task |

**Cycle**: For every task: run `/test` first to create the test suite from the spec, then `/implement` to write code that passes those tests. The agent checks off subtasks in `docs/tasks.md` as tests pass, linking each to the implementation file. Repeat per task until the milestone is complete.

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
                                                             │     ↓ checks off tasks.md
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
5. **GitHub references**: When `/github-init` runs (or is invoked by `/github-create-pr` or `/github-sync`), each milestone, task, and subtask gets a `GitHub:` field linking to its issue or milestone number.
6. **PR references**: When `/github-create-pr` (or `/github-sync` invoking it) detects newly-completed subtasks, it groups them under the parent task and opens one PR per task. The `PR:` field is added inline for the parent task and each subtask included in that PR.
7. **The `/implement` command reads `docs/tasks.md` before writing any code** to know what to build and to verify all prior subtasks in the current task are complete.
8. **The `/implement` command writes to `docs/tasks.md` after passing tests** to check off subtasks and add implementation references.
9. **Iterative updates**: If `/design` is re-run to refine the spec, `docs/tasks.md` is regenerated or updated to reflect changes. New items are appended; completed items are preserved.
10. **Integration awareness**: Subtasks that involve external services reference the Integration Registry. The `/implement` command consults both `docs/tasks.md` and `docs/integration-registry.md` when working on these subtasks.

### Example Structure

```markdown
## M1 — User Authentication
- Status: IN PROGRESS
- GitHub: milestone #1

### T1.1 — Login Flow
- Status: COMPLETE
- GitHub: issue #5
- PR: #42

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
- Integration registry enforcement rules
- Task tracker standards and status rules
- CI/CD considerations for team workflows

---

## Key Principles

1. **No ambiguity reaches code.** Every question the agent asks you is designed to resolve a decision that would otherwise become a bug or a refactor.
2. **Test-first, always.** Tests are written from the spec, not from the implementation. Code is written to satisfy tests.
3. **Configuration before code.** Every external service, API key, database connection, and environment variable is documented in the Integration Registry before any implementation begins. The agent never invents configuration on the fly.
4. **Task list is the source of truth.** The agent reads `docs/tasks.md` before writing code, checks off subtasks as tests pass, and links every completed subtask to its implementation file.
5. **Documentation is a living artifact.** The `/docs-sync` command timestamps and audits every change. Spec drift is caught, not ignored.
6. **Clean architecture is non-negotiable.** The architecture phase produces the folder structure, and that structure enforces separation of concerns regardless of project type.
7. **Commands are independent but aware.** You can run any command at any time, but the agent will warn you if prerequisite artifacts are missing and recommend the correct order.
