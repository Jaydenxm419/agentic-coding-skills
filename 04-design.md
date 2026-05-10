# /design — UI/UX & Component Design + Task List Generation

> **Purpose**: Define every user-facing screen, interaction flow, component specification, and state transition. Then generate the hierarchical task list (`docs/tasks.md`) that decomposes the entire project into Milestones → Tasks → Subtasks. The design spec and task list together form the actionable plan that drives implementation.
> **Prerequisites**: `docs/architecture.md` must exist and be signed off. If missing or unsigned, warn and recommend running `/architecture` first.
> **Output**: `docs/design.md` AND `docs/tasks.md`
> **Iterative**: Yes — loops through design refinement until all flows are complete, the task list is reviewed, and the developer signs off.

---

## Instructions for the Agent

You are creating the design specification and the project task list. Read `docs/project-init.md`, `docs/requirements.md`, and `docs/architecture.md` before starting.

**Behavior rules:**
1. If this project has a UI (web, mobile, desktop), walk through every screen. If it is an API/CLI/library, walk through every interface contract and developer experience flow.
2. For UI projects: ask if the developer has existing Google Stitch mockups or wireframes to reference. Incorporate their design decisions.
3. Present designs iteratively — one flow at a time. Get confirmation before moving to the next.
4. Every design decision must trace to a requirement AND have a home in the architecture.
5. State management and data flow must be specified per component, not left to implementation.
6. After the design spec is complete, generate the full task list before requesting sign-off.
7. This is NOT the last planning phase — `/integrations` follows. But all UI/UX and component decisions must be finalized here.

---

## Design Phase 1 — Navigation & Information Architecture

### For UI Projects:
Ask the developer:

1. **What are the top-level pages or screens?** (List every distinct view)
2. **What is the navigation structure?** (Sidebar, top nav, bottom tabs, breadcrumbs)
3. **What is the entry point?** (Landing page, login screen, onboarding wizard, dashboard)
4. **Are there Google Stitch mockups or wireframes already created?** If yes, ask the developer to describe each screen or share the mockup files for reference.

### For API / CLI / Library Projects:
Ask the developer:

1. **What are the primary interfaces a developer interacts with?** (endpoints, commands, exported functions)
2. **What is the developer experience flow?** (install → configure → first API call → common workflows)
3. **What does the help output or documentation landing page look like?**

---

## Design Phase 2 — Screen-by-Screen / Flow-by-Flow Specification

For each screen (UI) or interface (API/CLI), define:

1. **Purpose**: What does the user accomplish here?
2. **Layout**: Component hierarchy and spatial arrangement
3. **Components**: Every interactive element with its behavior
4. **State**: What data is loaded, how it changes, where it comes from (map to architecture data layer)
5. **Actions**: Every user action and its result (success path + error path)
6. **Transitions**: Where can the user go from here? What triggers the transition?
7. **Requirement traceability**: Which requirements does this screen/flow satisfy?

Present one flow at a time. Wait for developer feedback before proceeding.

---

## Design Phase 3 — Component Inventory

After all flows are specified, compile a master component inventory:

| Component | Used In | Props / Inputs | State | Architecture Layer |
|-----------|---------|----------------|-------|-------------------|
| [name] | [screens/flows] | [data it receives] | [internal state] | [presentation/application/domain] |

---

## Design Phase 4 — Interaction & State Flows

For complex interactions, document:

1. **State machines**: For components with multiple states (e.g., form → submitting → success → error)
2. **Data flow diagrams**: How data moves from user input to backend and back
3. **Optimistic updates**: Where the UI updates before server confirmation (if applicable)
4. **Loading and error states**: Every component that loads data must have defined loading and error UI

---

## Design Phase 5 — Task List Generation

**This is a critical step.** After the design spec is complete, generate `docs/tasks.md` — the hierarchical task list that decomposes the entire project into actionable work items.

### How to Generate the Task List

1. **Read all three spec documents**: `docs/requirements.md`, `docs/architecture.md`, `docs/design.md`
2. **Identify milestones**: Group related functionality into milestones. Each milestone represents a shippable chunk of the project (e.g., "User Authentication", "Dashboard", "API Integration").
3. **Break milestones into tasks**: Each task is a coherent unit of work within a milestone (e.g., "Login Flow", "Session Management", "Password Reset").
4. **Break tasks into subtasks**: Each subtask is a single, testable, implementable unit of work (e.g., "Email/password validation logic", "JWT token generation", "Login API endpoint").
5. **Ensure full traceability**: Every requirement from `docs/requirements.md` must map to at least one subtask. If a requirement has no subtask, create one. Present any gaps to the developer.
6. **Flag integration-dependent subtasks**: For any subtask that will require an external service (database query, API call, auth provider), note it with an `Integration:` field. These will be validated against the Integration Registry after `/integrations` runs.

### Task List Rules (from `_standards.md` Section 6)

- **ID format**: `M{n}` for milestones, `T{m}.{n}` for tasks, `S{m}.{n}.{p}` for subtasks
- **Statuses**: `NOT STARTED`, `IN PROGRESS`, `COMPLETE`, `BLOCKED`
- **Strict rollup**: Parent status is derived from children. No manual overrides.
- **Every subtask must reference**: The requirement(s) it satisfies
- **Test and Impl fields**: Initialized as `(pending /test)` and `(pending /implement)` — filled in during Phase 2

### Task List Output Format

Generate `docs/tasks.md` using this structure:

```markdown
# Project Task List

> Generated: [timestamp]
> Derived from: docs/requirements.md, docs/architecture.md, docs/design.md
> Last updated: [timestamp]

## Summary
| Metric | Count |
|--------|-------|
| Milestones | [n] |
| Tasks | [n] |
| Subtasks | [n] |
| Completed | 0 |
| Remaining | [n] |

## Traceability Check
| Requirement | Mapped Subtask(s) | Status |
|-------------|-------------------|--------|
| [REQ description] | S1.1.1, S1.2.3 | ✅ Covered |
| [REQ description] | (none) | ❌ Gap — needs subtask |

---

## M1 — [Milestone Name]
- Status: NOT STARTED
- Description: [what this milestone delivers]

### T1.1 — [Task Name]
- Status: NOT STARTED
- Requirement refs: [which requirements this satisfies]

- [ ] S1.1.1 — [Subtask description]
  - Requirement: [specific requirement or acceptance criterion]
  - Integration: [service name, if applicable — validated after /integrations]
  - Test: (pending /test)
  - Impl: (pending /implement)

(continue for all subtasks, tasks, milestones)
```

---

## Design Phase 6 — Review & Sign-Off

Present both `docs/design.md` and `docs/tasks.md` to the developer. Walk through:

1. **Design completeness**: Are all screens/flows specified with zero ambiguity?
2. **Task list review**: Are the milestones logically grouped? Are subtasks granular enough to be individually testable? Is every requirement covered?
3. **Sign-off checklist**: Present the Design Sign-Off checklist from `_standards.md`.

When the developer signs off, append to `docs/design.md`:

```markdown
## Sign-Off
- Signed off by: [developer name or "Developer"]
- Date: [timestamp]
- Task list generated: docs/tasks.md
- Open items: [none, or list]
```

---

## Post-Sign-Off

Tell the developer:
1. **Files created/updated**: `docs/design.md` and `docs/tasks.md`
2. **Next command**: `/integrations` — to catalog all external services, environment variables, and connection details before implementation begins
3. **Optional now**: Run `/github-init` to push the new task list to GitHub as milestones, issues, and sub-issues. This is the project's initial scaffolding — it works from any branch (including the default branch) because no code is committed at this stage. Only the static GitHub state is created. This can be done now or after `/integrations` (recommended after, in case integration setup adds new subtasks).
4. **Reminder**: The task list flagged [n] subtasks as integration-dependent. The `/integrations` phase will validate that every external service has a registry entry. After `/integrations` sign-off, the hard gate lifts.

Ask: "Would you like to run `/github-init` now to push these tasks to GitHub, or wait until after `/integrations`?"
