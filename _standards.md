# Standards & Validation Rules

> This file is referenced by every command in the workflow. It defines the non-negotiable quality standards that apply to all project types. When any command checks for compliance, it checks against these rules.

---

## 1. Prerequisites Check

Before executing any command, the agent MUST verify the following and warn the user if any are missing:

```
PREREQUISITE_ARTIFACTS:
  /init           → None (always runnable)
  /requirements   → docs/project-init.md must exist
  /architecture   → docs/requirements.md must exist and be signed off
  /design         → docs/architecture.md must exist and be signed off
  /integrations   → docs/design.md must exist and be signed off
  /test           → docs/architecture.md, docs/design.md, docs/integration-registry.md,
                     AND docs/tasks.md must exist and be signed off
  /implement      → docs/architecture.md, docs/design.md, docs/integration-registry.md,
                     AND docs/tasks.md must exist and be signed off
                     AND relevant test files must exist for the target module
                     AND the target task's Human Setup checklist must be fully
                         confirmed ([x] on every item) before code generation
                         (HARD GATE — see /implement Step 2)
  /docs-sync      → At least one implementation file must exist
  /audit          → docs/ folder must contain at least requirements, architecture,
                     design, tasks, and integration-registry specs
  /github-init    → docs/tasks.md must exist
                     gh CLI must be installed and authenticated
                     Current directory must be a git repo (remote optional —
                     skill walks through remote setup if missing)
  /github-create-pr → docs/tasks.md must exist
                     gh CLI must be installed and authenticated
                     Current directory must be a git repo with a GitHub remote
                     Current branch must NOT be the default branch
  /github-sync    → docs/tasks.md must exist
                     gh CLI must be installed and authenticated
                     Current directory must be a git repo with a GitHub remote
                     (Branch rules apply only to PR-creation portions of the run)
```

If prerequisites are missing, the agent MUST:
1. List exactly which artifacts are missing
2. Recommend which command to run first
3. Ask the user if they want to proceed anyway (override) or follow the recommended order

**Note**: The Human Setup checklist gate on `/implement` is the ONE prerequisite with no override path. Even if every other artifact exists and is signed off, `/implement` will refuse to generate code until every item in the target task's `Human Setup:` field is `[x]`-confirmed by the developer in chat.

---

## 2. Sign-Off Protocol

Any iterative phase (requirements, architecture, design, integrations) follows this protocol:

### Completion Checklist
The agent maintains an internal checklist of unresolved items. Before requesting sign-off, ALL of the following must be true:

- [ ] Every functional requirement has at least one acceptance criterion
- [ ] Every external dependency is identified with version or API contract
- [ ] Every user-facing flow has defined entry point, happy path, and error states
- [ ] Every data entity has defined shape/schema
- [ ] Every external service is cataloged in the Integration Registry (after /integrations)
- [ ] Every environment variable is named, scoped, and mapped to its service
- [ ] No open questions remain (all "TBD" items are resolved)
- [ ] No contradictions exist between sections of the spec

### Sign-Off Process
1. Agent presents the completed checklist summary
2. Agent explicitly asks: "Do you sign off on this phase? (yes / no / list concerns)"
3. If no → agent asks targeted follow-up questions to resolve concerns
4. If yes → agent writes the sign-off timestamp and user confirmation to the spec file
5. Phase is marked complete and the next phase is unlocked

---

## 3. Code Quality Standards

These apply during `/test`, `/implement`, and `/audit` phases.

### Naming Conventions
- Files: kebab-case (e.g., `user-service.ts`, `auth-controller.py`)
- Classes/Components: PascalCase (e.g., `UserService`, `AuthController`)
- Functions/Methods: camelCase or snake_case per language convention
- Constants: SCREAMING_SNAKE_CASE (e.g., `MAX_RETRY_COUNT`)
- Database tables: snake_case plural (e.g., `user_accounts`)
- API endpoints: kebab-case (e.g., `/api/v1/user-profiles`)
- Environment variables: SCREAMING_SNAKE_CASE with service prefix (e.g., `STRIPE_SECRET_KEY`)
- Test files: Mirror source file names with `.test` or `.spec` suffix

### Error Handling Requirements
Every module MUST implement:
1. **Input validation** — Validate all external inputs at the boundary (API layer, UI layer, CLI layer)
2. **Typed errors** — Use custom error types/classes, never throw generic strings
3. **Error propagation** — Errors bubble up through defined channels (Result types, error middleware, error boundaries)
4. **Logging** — All caught errors log: timestamp, error type, context (which module, which operation, which input)
5. **User-facing messages** — Never expose stack traces or internal error details to end users
6. **Graceful degradation** — Define fallback behavior for every external dependency failure

### Documentation Requirements
Every module MUST have:
1. **Module header comment** — Purpose, dependencies, public API summary
2. **Public function/method docs** — Parameters, return types, thrown errors, usage example
3. **Complex logic comments** — Any non-obvious algorithm or business rule gets an inline explanation
4. **No orphan code** — Every function is either called or explicitly marked as a public API entry point

---

## 4. Clean Architecture Enforcement

### Layer Rules
The architecture phase defines the project's layers. Regardless of specific layer names, these rules apply:

1. **Domain/Core layer** — ZERO external dependencies. No framework imports, no database drivers, no HTTP libraries.
2. **Application/Use Case layer** — Depends only on domain. Orchestrates business logic. Communicates with external services ONLY through interfaces/ports defined here.
3. **Infrastructure/Adapter layer** — Implements interfaces defined by the application layer. This is the ONLY layer that directly references external services, SDKs, and connection details.
4. **Presentation layer** — Depends on application layer. Handles UI rendering, API response formatting, CLI output.

### Dependency Direction
Dependencies ALWAYS point inward (presentation → application → domain). Never outward. The agent will flag any import that violates this direction.

### External Service Access Rule
**No module outside the infrastructure layer may directly reference environment variables, API keys, or external service SDKs.** All external access goes through interfaces defined in the application layer and implemented in the infrastructure layer. The Integration Registry (`docs/integration-registry.md`) documents which infrastructure adapters connect to which services.

---

## 5. Integration Registry Enforcement

These rules apply during `/implement`, `/test`, `/docs-sync`, and `/audit` phases.

### During Implementation (`/implement`)
1. Before writing any module that connects to an external service, the agent MUST consult `docs/integration-registry.md`
2. The agent MUST use the exact environment variable names from the registry — never invent new ones
3. The agent MUST use the documented authentication method — never improvise
4. If implementation reveals a NEW external dependency not in the registry, the agent MUST:
   - Stop implementation on that module
   - Warn the developer: "This module requires [service] which is not in the Integration Registry"
   - Recommend running `/integrations` to update the registry
   - Ask if the developer wants to update now or override
5. Before implementation begins for any target task, the agent MUST run the Human Intervention Pre-Flight Check defined in `/implement` Step 2 and obtain developer confirmation on every checklist item. This is a hard gate with no overrides. The checklist draws from the Integration Registry, the test files for the target task, and the architecture/design specs — any non-code setup (account creation, API key generation, webhook configuration, local services, IAM grants, etc.) must be confirmed complete before code is written.
6. If implementation reveals a missed human-setup item mid-task (e.g., a test fails because a service isn't actually running despite being marked `[x]`), the agent MUST stop, add the item back to the `Human Setup:` checklist as `[ ]`, and request reconfirmation before resuming.

### During Testing (`/test`)
1. Test configuration MUST include mock/stub values for every required environment variable in the registry
2. Integration tests MUST reference the registry for expected service behavior
3. The test setup MUST validate that all required env vars are present before running (fail fast with clear messages, not cryptic connection errors)

### During Doc Sync (`/docs-sync`)
1. Compare the `.env.example` file against the Integration Registry and flag discrepancies
2. Check that no environment variable exists in code that is not in the registry
3. Check that no registry entry is missing from the codebase
4. Verify that the naming convention is consistently applied

### During Audit (`/audit`)
1. Include an "Integration Health" section in every audit report
2. Verify all registry entries are still used in code (flag dead integrations)
3. Verify no undocumented environment variables exist (flag rogue variables)
4. Verify `.env.example` is in sync with the registry
5. Verify no secrets are hardcoded anywhere in the codebase
6. Verify all external service access goes through the infrastructure layer

---

## 6. Task Tracker Standards

These rules apply to `docs/tasks.md` and govern how every command interacts with the task list.

### ID Format
- Milestones: `M{n}` (e.g., M1, M2, M3)
- Tasks: `T{m}.{n}` (e.g., T1.1, T1.2, T2.1)
- Subtasks: `S{m}.{n}.{p}` (e.g., S1.1.1, S1.1.2, S1.2.1)

### Statuses
| Status | Meaning |
|--------|---------|
| `NOT STARTED` | No subtasks have been started |
| `IN PROGRESS` | At least one subtask is complete but not all |
| `COMPLETE` | All subtasks are checked off with test and implementation references |
| `BLOCKED` | A subtask cannot proceed due to an external dependency — blocker must be described |

### Strict Rollup Rules
- A **task** status is derived exclusively from its subtasks. No manual override.
- A **milestone** status is derived exclusively from its tasks. No manual override.
- A task CANNOT be marked `COMPLETE` unless every subtask has: `[x]` checkbox, a `Test:` file path (not pending), and an `Impl:` file path (not pending).
- A milestone CANNOT be marked `COMPLETE` unless every task within it is `COMPLETE`.

### Implementation References
When a subtask is completed by `/implement`, the agent records:
- **Test**: Path to the test file (e.g., `tests/auth/login.test.ts`)
- **Impl**: Path to the primary implementation file (e.g., `src/auth/validators.ts`)
- File paths are relative to the project root
- If implementation spans multiple files, record the main entry point file

### Human Setup Field
The `Human Setup:` field is added to a task by `/implement` Step 2 before any code is written. It captures all non-code prerequisites that the developer must complete manually before implementation can succeed.

- The field lives on the task (not on individual subtasks), since human setup typically applies across all subtasks in a task
- Items are written as concrete, action-oriented statements, each tagged with their source (Registry / Tests / Architecture)
- `[ ]` = pending; `[x]` = confirmed complete by the developer
- Each confirmed item should include a confirmation timestamp inline: `[x] [item] — confirmed YYYY-MM-DD HH:MM`
- The field is preserved on completed tasks as a historical record — do NOT delete it after a task is COMPLETE
- On re-runs of `/implement`, the agent merges new items with existing ones rather than overwriting (preserves prior `[x]` confirmations)

### Traceability
- Every subtask MUST reference the requirement(s) it satisfies from `docs/requirements.md`
- Every requirement MUST map to at least one subtask — gaps are flagged during `/design`
- Subtasks involving external services MUST note the integration (e.g., "Integration: Stripe — see integration-registry.md")

### Task List Lifecycle
| Phase | How it interacts with `docs/tasks.md` |
|-------|---------------------------------------|
| `/design` | **Generates** the task list from requirements + architecture + design specs |
| `/integrations` | May **append** integration-specific subtasks if external service setup requires implementation work |
| `/test` | **Reads** the task list to know what to test; **writes** test file paths to subtask `Test:` fields |
| `/implement` | **Reads** the task list before coding; **writes** the `Human Setup:` checklist on the target task in Step 2; **writes** `[x]`, `Impl:` paths, and status rollups after tests pass |
| `/docs-sync` | **Validates** task list integrity — stale file refs, rollup errors, orphaned subtasks, and stale `Human Setup:` items that may no longer be applicable |
| `/audit` | **Audits** full task list — completeness, traceability, file existence, status accuracy, and the integrity of `Human Setup:` records on completed tasks |
| `/github-init` | **Reads** the task list and **writes** `GitHub:` field references after creating milestones, issues, and sub-issues |
| `/github-create-pr` | **Reads** the task list to find newly-completed work, **writes** `PR:` field references after opening PRs (auto-invokes `/github-init` logic for missing GitHub items) |
| `/github-sync` | **Reads** the task list, identifies all gaps, and delegates to `/github-init` and `/github-create-pr` to fill them. **Writes** both `GitHub:` and `PR:` references as a result. |

### GitHub Integration Standards
The three GitHub commands (`/github-init`, `/github-create-pr`, `/github-sync`) follow a consistent set of rules:

- **Title format**: Every issue, milestone, and PR title MUST start with the relevant ID (e.g., `M1 — User Authentication`, `T1.1 — Login Flow`, `S1.1.1 — Email/password validation logic`). PRs use the parent task ID, or comma-separated IDs if multiple tasks are covered.
- **Labels**: Every issue and PR receives `workflow:agentic` plus its hierarchy labels (`milestone:M1`, `task:T1.1`)
- **Issue body sections**: Every issue body must contain Task, Summary, Implementation Strategy, Important Reminders, and Configuration Instructions (when integration-related)
- **PR body sections**: Every PR body must contain Summary, Subtasks Completed, Parent Task, Test Plan, Configuration Instructions (when applicable), Files Changed, Linked Issues (with `Closes #N` keywords), and a footer noting human review is required
- **State mirror**: Subtasks marked `[x]` in `docs/tasks.md` correspond to closed GitHub issues; unchecked subtasks correspond to open issues. When a subtask has implementation work AND a PR is linked, closure is deferred to PR merge via `Closes #N`.
- **Idempotency**: All three commands are safe to re-run. They read existing `GitHub:` and `PR:` references and update or skip rather than duplicate.
- **Source of truth**: `docs/tasks.md` is canonical. Status changes in GitHub are overwritten on next sync — developers update `docs/tasks.md` (or use `/implement`) to change real status. PR merging closes issues; reverting `[x]` to `[ ]` in tasks.md will reopen the issue and flag the corresponding open PR for review.
- **No auto-merging**: NONE of the three commands ever merge a PR. PRs require human review before merge. Auto-merge is never enabled.

### Command-Specific Rules

**`/github-init`** — branch-agnostic, no git operations:
- Performs zero git operations. Only calls the GitHub REST API via `gh`.
- Works from any branch including the default branch — no risk of unsafe pushes.
- The single exception: during initial remote-setup walkthrough (when no remote exists), the skill may run `git push` ONCE with explicit developer confirmation.

**`/github-create-pr`** — strict-branch-confined:
- Operates STRICTLY on the current branch — never switches branches or auto-creates them.
- Refuses to commit or open a PR when the current branch is the repository's default branch.
- Auto-invokes `/github-init` logic for any milestone/task/subtask that doesn't yet have a `GitHub:` reference. The skill is standalone — `/github-init` is not required to have run separately.
- Branch creation is only permitted when the developer explicitly requests it during the conversation.

**`/github-sync`** — long-running reconciliation:
- Walks the entire `docs/tasks.md` hierarchy and identifies every gap (missing GitHub items, out-of-sync items, missing PRs, stale PRs, orphan PRs, state mismatches).
- Reports all gaps before acting. Asks the developer which categories to reconcile.
- Delegates: invokes `/github-init` logic for missing GitHub items, `/github-create-pr` logic for missing PRs.
- Branch rules from `/github-create-pr` apply only to the PR-creation categories. The issue/milestone scaffolding categories work from any branch.

---

## 7. Test Standards

### Test-First Rule
Tests are ALWAYS written before implementation code. The `/test` command generates tests from specs. The `/implement` command writes code to pass those tests. Never the reverse.

### Test Types Required (where applicable)
1. **Unit tests** — Every pure function, every domain entity method
2. **Integration tests** — Every adapter that connects to an external service (using mocks/stubs based on the Integration Registry)
3. **Contract tests** — Every API endpoint matches its documented contract from `docs/architecture.md`
4. **Acceptance tests** — At least one per functional requirement, testing the full user flow

### Test Naming Convention
```
[unit|integration|contract|acceptance]-[module-name].[test-extension]

Examples:
  unit-user-service.test.ts
  integration-stripe-adapter.test.ts
  contract-auth-api.test.py
  acceptance-checkout-flow.spec.ts
```

---

## 8. CI/CD Considerations (If Applicable)

If the project uses CI/CD (from `docs/project-init.md`):

1. **Environment variable injection** — CI/CD pipeline must inject variables per environment as documented in the Integration Registry
2. **Secret management** — Pipeline must pull secrets from the configured secret manager (from Integration Registry Phase 1)
3. **Test stage** — Must run all test suites and fail the build on any failure
4. **Lint stage** — Must check code quality standards from Section 3
5. **Deploy stage** — Must match the deployment strategy from `docs/architecture.md`
6. **Integration validation** — Post-deploy health checks should verify external service connectivity for all critical integrations documented in the registry
