# /docs-sync — Documentation Synchronization

> **Purpose**: Detect drift between the codebase and the specification documents. Generate a timestamped audit report, fix discrepancies, and update `docs/tasks.md` if scope has changed. Also validates `.env.example` against the Integration Registry.
> **Prerequisites**: `docs/` folder must contain at least `requirements.md`, `architecture.md`, `design.md`, `tasks.md`, and `integration-registry.md`. If missing, warn and recommend completing the planning phases first.
> **Output**: Timestamped audit report in `docs/audits/` + updated spec documents and task list.
> **Iterative**: Yes — run after any significant codebase change.

---

## Instructions for the Agent

You are synchronizing documentation with the current state of the codebase. Read ALL documents in `docs/` and then analyze the codebase for drift.

**Behavior rules:**
1. Be thorough but focused on drift — this is not a full audit (use `/audit` for that).
2. Every discrepancy must be categorized and actionable.
3. Never silently update specs — present findings to the developer and get confirmation before making changes.
4. If implementation has introduced new scope, add new subtasks to `docs/tasks.md`.
5. If implementation has deviated from the spec, flag it — the developer decides whether to update the spec or fix the code.

---

## Step 1 — Generate Audit Report

Create a timestamped audit file:

**Filename format**: `docs/audits/sync-YYYY-MM-DD-HHMMSS.md`

**Metadata block**:
```markdown
# Documentation Sync Report

> Generated: [full timestamp]
> Trigger: [manual / post-implementation / post-refactor]
> Scope: [what was checked — full codebase or specific modules]
> Previous sync: [filename of last sync report, or "none"]
```

---

## Step 2 — Drift Detection

Compare each spec document against the codebase:

### Architecture Drift
- [ ] Folder structure matches `docs/architecture.md`
- [ ] Component boundaries are respected (no cross-layer imports violating clean architecture)
- [ ] Data models in code match schemas in spec
- [ ] API endpoints in code match contracts in spec
- [ ] New files or modules exist that are not documented

### Design Drift
- [ ] UI components match `docs/design.md` component inventory
- [ ] State management follows the design spec
- [ ] Navigation/routing matches the spec
- [ ] New screens or flows exist that are not documented

### Task List Drift
- [ ] All completed subtasks in `docs/tasks.md` have valid `Impl:` file paths (files still exist)
- [ ] No implemented features exist that are not tracked in `docs/tasks.md`
- [ ] Status rollups are correct (task/milestone statuses match their children)
- [ ] No orphaned subtasks (subtask exists but its implementation file was deleted or moved)
- [ ] If any `GitHub:` references exist, the `gh` CLI can confirm the referenced issues/milestones still exist (run `gh issue view {n}` for each reference)
- [ ] If subtasks have been added since last `/github-init` or `/github-sync` run, flag them as needing a GitHub push

### Integration Registry Drift
- [ ] `.env.example` variables match `docs/integration-registry.md`
- [ ] No environment variables in code are missing from the registry
- [ ] No registry entries are missing from the codebase
- [ ] Naming convention is consistently applied
- [ ] No secrets are hardcoded in the codebase

### Requirements Drift
- [ ] All requirements in `docs/requirements.md` still have corresponding subtasks
- [ ] No implemented features contradict stated requirements

---

## Step 3 — Categorize Findings

Classify each discrepancy:

| Category | Description | Action |
|----------|-------------|--------|
| **Spec behind code** | Code has evolved beyond what the spec describes | Update spec to match code |
| **Code behind spec** | Spec describes something not yet implemented | Verify it's still in scope, update `docs/tasks.md` |
| **Contradiction** | Code contradicts the spec | Developer decides: fix code or update spec |
| **Orphan** | Code or spec item exists with no counterpart | Remove or document |
| **Task list stale** | Implementation references point to moved/deleted files | Update file paths in `docs/tasks.md` |
| **Registry drift** | Env vars in code don't match registry | Update registry or fix code |
| **GitHub drift** | New subtasks not yet pushed, or referenced issues no longer exist | Recommend running `/github-init` (for new items) or `/github-sync` (for full reconciliation) |

---

## Step 4 — Present Findings & Apply Fixes

Present all findings to the developer in the audit report, grouped by category. For each finding, propose the specific fix. Wait for developer confirmation before applying.

After applying fixes, update spec documents with changelog entries and update `docs/tasks.md` if subtask paths, statuses, or new items changed.

---

## Step 5 — Completion

Tell the developer:
1. **Audit location**: Path to the generated sync report
2. **Specs updated**: List which spec files were modified
3. **Task list changes**: List any additions, removals, or path updates to `docs/tasks.md`
4. **Integration registry changes**: List any env var additions or removals
5. **GitHub sync needed?**: If new subtasks were added or `GitHub:` references became stale, recommend running `/github-init` (for new items) or `/github-sync` (for full reconciliation including missing PRs). If completed work has no `PR:` references, recommend `/github-create-pr`.
6. **Unresolved items**: List any discrepancies deferred for later
7. **Health summary**: Overall alignment score (e.g., "47 out of 50 subtasks are aligned")
8. **Next recommended command**: Usually `/test` + `/implement` for uncovered items, `/github-init` for newly-added scope, `/github-create-pr` for completed work, `/github-sync` for full reconciliation, or `/audit` for a deeper review
