# /audit — Codebase Audit & Health Check

> **Purpose**: Perform a deep, comprehensive review of the entire codebase against the specification, clean architecture rules, test coverage, code quality standards, task completion, and integration health. This is a periodic health check — more thorough than `/docs-sync`, which focuses on incremental drift.
> **Prerequisites**: `docs/` folder must contain at least `requirements.md`, `architecture.md`, `design.md`, `tasks.md`, and `integration-registry.md`. If missing, warn and recommend completing the planning phases first.
> **Output**: Timestamped audit report in `docs/audits/`.
> **Iterative**: Yes — run periodically (recommended: before major releases, after large refactors, monthly for active projects).

---

## Instructions for the Agent

You are performing a full codebase audit. Read ALL spec documents and then systematically analyze the entire codebase against every standard defined in `_standards.md`.

**Behavior rules:**
1. This is a thorough, critical review. Do not gloss over issues to be polite.
2. Every finding must reference the specific standard, spec section, or best practice being violated.
3. Categorize findings by severity and affected area.
4. Provide actionable recommendations for every finding.
5. Generate the full audit report before recommending changes.

---

## Audit Dimension 1 — Clean Architecture Compliance

- [ ] **Domain isolation**: Does the domain layer import ANYTHING from infrastructure, presentation, or framework-specific packages?
- [ ] **Dependency direction**: Do all dependencies point inward (presentation → application → domain)?
- [ ] **Interface boundaries**: Does infrastructure implement interfaces defined in domain/application?
- [ ] **No circular dependencies**: Run a dependency analysis across all modules.
- [ ] **Folder structure**: Does the actual file tree match `docs/architecture.md`?
- [ ] **External service access**: All external access goes through infrastructure layer only.

---

## Audit Dimension 2 — Spec Traceability Matrix

Build a complete traceability matrix:

| Requirement | Subtask(s) | Test(s) | Implementation | Status |
|-------------|-----------|---------|----------------|--------|
| [from requirements.md] | [from tasks.md] | [test files] | [impl files] | ✅ / ❌ / ⚠️ |

Flag: requirements with no subtasks (coverage gap), subtasks with no tests (testing gap), subtasks marked complete but tests fail (regression), implemented code with no tracing requirement (scope creep).

---

## Audit Dimension 3 — Task List Integrity

Validate `docs/tasks.md` thoroughly:

- [ ] All IDs follow the `M{n}`, `T{m}.{n}`, `S{m}.{n}.{p}` format
- [ ] All status rollups are correct (recalculate from subtask checkboxes)
- [ ] All `[x]` subtasks have a non-pending `Test:` path
- [ ] All `[x]` subtasks have a non-pending `Impl:` path
- [ ] All referenced test files actually exist on disk
- [ ] All referenced implementation files actually exist on disk
- [ ] The Summary table counts match the actual counts
- [ ] The Traceability Check table is accurate
- [ ] All subtasks with `Integration:` fields reference valid registry entries
- [ ] All `GitHub:` references (if any) point to issues/milestones that exist in the connected repo
- [ ] GitHub issue states match `docs/tasks.md` statuses (closed for `[x]`, open for `[ ]`)

---

## Audit Dimension 4 — Integration Health

Validate `docs/integration-registry.md` against the codebase:

- [ ] All registry entries are still used in code (flag dead integrations)
- [ ] No undocumented environment variables exist in code (flag rogue variables)
- [ ] `.env.example` is in sync with the registry
- [ ] No secrets are hardcoded anywhere in the codebase
- [ ] All external service access goes through the infrastructure layer
- [ ] Naming convention is consistently applied to all env vars
- [ ] Every integration-dependent subtask in `docs/tasks.md` has a matching registry entry

---

## Audit Dimension 5 — Test Coverage Analysis

- [ ] Run the project's test suite and capture results
- [ ] Calculate coverage metrics (statement, branch, function)
- [ ] Identify untested public functions or endpoints
- [ ] Verify that every subtask's test file has at least one passing test (for completed subtasks)
- [ ] Check for test quality: Are tests actually asserting behavior or just running without assertions?
- [ ] Verify integration test mocks align with current registry documentation

---

## Audit Dimension 6 — Code Quality

- [ ] **Error handling**: Scan for empty catch blocks, untyped throws, silent failures
- [ ] **Naming conventions**: Verify compliance with `_standards.md` naming rules
- [ ] **Documentation**: Check for missing docstrings on public APIs
- [ ] **Dead code**: Identify unused exports, unreachable branches, commented-out code
- [ ] **Dependency health**: Check for deprecated packages, security vulnerabilities, unused dependencies

---

## Audit Dimension 7 — CI/CD Health

(If applicable based on `docs/project-init.md`)

- [ ] Build passes cleanly
- [ ] All tests pass in CI
- [ ] Deployment pipeline is functional
- [ ] Environment configuration is documented
- [ ] Environment variable injection matches the Integration Registry

---

## Audit Report Format

**Filename**: `docs/audits/audit-YYYY-MM-DD-HHMMSS.md`

```markdown
# Codebase Audit Report

> Generated: [full timestamp]
> Audit type: Full codebase audit
> Previous audit: [filename, or "none"]
> Project: [name from project-init.md]

## Executive Summary
- Overall health: [Healthy / Needs Attention / Critical]
- Architecture violations: [count]
- Test coverage: [percentage]
- Task completion: [X of Y subtasks — Z%]
- Integration health: [all healthy / n issues]
- Open issues: [count by severity]

## 1. Clean Architecture Compliance
[violations table with file, violation, severity, fix]

## 2. Spec Traceability Matrix
[full matrix]

## 3. Task List Integrity
[findings — ID errors, stale references, rollup mismatches]

## 4. Integration Health
[registry vs codebase comparison, rogue vars, dead integrations, hardcoded secrets]

## 5. Test Coverage Analysis
[coverage table and gaps]

## 6. Code Quality
[findings by category]

## 7. CI/CD Health
[pipeline analysis]

## Recommended Actions
| Priority | Action | Severity | Affected Files | Task Ref |
|----------|--------|----------|----------------|----------|
| 1 | [most critical fix] | Critical | [files] | [S-ID if applicable] |
| 2 | ... | High | ... | ... |

## Comparison with Previous Audit
(If a previous audit exists in docs/audits/, compare key metrics)
| Metric | Previous | Current | Trend |
|--------|----------|---------|-------|
| Architecture violations | [n] | [n] | [improved/declined/stable] |
| Test coverage | [%] | [%] | [improved/declined/stable] |
| Task completion | [%] | [%] | [improved/declined/stable] |
| Integration issues | [n] | [n] | [improved/declined/stable] |
```

---

## Post-Audit

Tell the developer:
1. **Audit location**: Path to the generated report
2. **Top 3 priorities**: The most impactful actions to take
3. **Task list issues** (if any): Stale references, rollup errors, or gaps found
4. **Integration issues** (if any): Rogue variables, dead integrations, hardcoded secrets
5. **GitHub sync issues** (if any): Mismatched issue states, missing pushes, broken references — recommend `/github-sync` for full reconciliation, or `/github-init` if only scaffolding is needed
6. **Next recommended command**: Usually `/docs-sync` to fix drift, `/github-sync` to realign GitHub state, or `/test` + `/implement` for missing coverage
7. **Schedule recommendation**: When to run the next audit
