# /requirements — Requirements Gathering

> **Purpose**: Conduct exhaustive requirements gathering through iterative Q&A. The output should be detailed enough that the architecture and design phases have zero ambiguity about what the project must do.
> **Prerequisites**: `docs/project-init.md` must exist. If missing, warn and recommend running `/init` first.
> **Output**: `docs/requirements.md`
> **Iterative**: Yes — loops through requirement categories until the developer signs off.

---

## Instructions for the Agent

You are gathering complete requirements for the project. Read `docs/project-init.md` before starting to understand the project type, stack, and audience.

**Behavior rules:**
1. Walk through each requirement category one at a time. Do not skip categories even if they seem inapplicable — ask the developer to confirm they are not relevant.
2. For each requirement, push for specificity. Vague requirements like "fast" or "secure" must be quantified (e.g., "< 200ms response time", "OWASP Top 10 compliance").
3. After completing all categories, present the full requirements document for review.
4. The developer must explicitly sign off. If they request changes, loop back and refine.
5. Reference `_standards.md` for quality expectations.

---

## Category 1 — Core Features

Ask the developer:

1. **What are the core features of this project?** List every distinct capability the system must have.
2. **For each feature**: What does success look like? What is the user's goal when using this feature?
3. **Are there any features that are explicitly out of scope?** (Important to document what the project will NOT do.)

---

## Category 2 — User Stories / Use Cases

For each core feature, ask:

1. **Who is the actor?** (end user, admin, API consumer, system, etc.)
2. **What action do they take?**
3. **What is the expected outcome?**
4. **What are the acceptance criteria?** (specific, testable conditions that must be true)

Format each as:
```
As a [actor], I want to [action] so that [outcome].
Acceptance criteria:
- [criterion 1]
- [criterion 2]
```

---

## Category 3 — Non-Functional Requirements

Ask about each, pushing for specific targets:

1. **Performance**: Response times, throughput, concurrent users
2. **Security**: Authentication method, authorization model, data encryption, compliance requirements
3. **Scalability**: Expected growth, horizontal/vertical scaling needs
4. **Reliability**: Uptime target, disaster recovery, backup strategy
5. **Accessibility**: WCAG level, screen reader support, keyboard navigation (if UI project)
6. **Internationalization**: Multi-language support, locale handling, RTL support
7. **Browser/platform support**: Minimum versions, mobile responsiveness

---

## Category 4 — Data Requirements

1. **What data does the system manage?** List every entity or data object.
2. **What are the relationships between entities?**
3. **Are there external data sources?** (APIs, imports, third-party integrations)
4. **What are the data retention and privacy requirements?**
5. **Are there any regulatory constraints on data handling?** (GDPR, HIPAA, SOC 2, etc.)

---

## Category 5 — Integration & Dependency Requirements

1. **What external services or APIs does this project integrate with?**
2. **Are there existing systems this must interoperate with?**
3. **Are there third-party libraries or tools that are required or prohibited?**
4. **What happens when an external dependency is unavailable?** (fallback behavior)

---

## Category 6 — Edge Cases & Error Scenarios

For each core feature, ask:

1. **What happens when the input is invalid?**
2. **What happens when an external service is down?**
3. **What happens under concurrent or duplicate operations?**
4. **What are the known edge cases?**
5. **What should the system explicitly reject?**

---

## Phase: Review & Sign-Off

After completing all categories, present the full document and the sign-off checklist from `_standards.md` (Section 2).

When the developer signs off, append:

```markdown
## Sign-Off
- Signed off by: [developer name or "Developer"]
- Date: [timestamp]
- Open items: [none, or list any deferred items]
```

---

## Post-Sign-Off

Tell the developer:
1. **File created/updated**: `docs/requirements.md`
2. **Next command**: `/architecture` — to design the system architecture based on these requirements
3. **Reminder**: Any changes to requirements after architecture begins should trigger a re-review of affected architecture decisions
