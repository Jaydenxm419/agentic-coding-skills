# GEMINI.md — Agentic Development Workflow

This directory contains a portable, repeatable blueprint for software development projects. It defines a structured, agent-friendly workflow that prioritizes clear requirements, robust architecture, and test-first implementation.

## Project Overview

The **Agentic Development Workflow** is a collection of Markdown-based instructions ("commands") designed to guide an AI agent (and developers) through the full lifecycle of a software project. It enforces a "planning-first" mentality where no code is written until requirements, architecture, and design are fully vetted and signed off.

### Core Principles
- **No ambiguity reaches code:** Decision-making happens in the planning phase.
- **Test-first, always:** Tests are derived from specs, and code is written to satisfy tests.
- **Task tracker as source of truth:** Progress is tracked in a central `docs/tasks.md` file.
- **Clean Architecture:** Separation of concerns is non-negotiable.
- **Integration Awareness:** External services are cataloged before use.

## Key Files

### Core Workflow Files
- **WORKFLOW.md:** The main entry point and overview of the entire process. It describes the phases, command order, and the task tracker structure.
- **_standards.md:** Defines the non-negotiable quality standards, naming conventions, error handling requirements, and clean architecture rules that apply across all phases.

### Command Files (Planning Phase)
- **01-init.md (`/init`):** Establishes project identity, type detection, and initial tooling.
- **02-requirements.md (`/requirements`):** Iterative process for exhaustive requirements gathering.
- **03-architecture.md (`/architecture`):** Defines system design, folder structure, and component boundaries.
- **04-design.md (`/design`):** Specifies UI/UX flows, component specs, and API contracts. Also generates `docs/tasks.md`.
- **05-integrations.md (`/integrations`):** Catalogs external services, environment variables, and connection details.

### Command Files (Execution Phase)
- **06-test.md (`/test`):** Generates test suites based on the signed-off specs.
- **07-implement.md (`/implement`):** Guides the implementation of code that passes the generated tests.
- **08-docs-sync.md (`/docs-sync`):** Detects and resolves drift between code and documentation.
- **09-audit.md (`/audit`):** Performs deep codebase health checks against the specs.

### Command Files (GitHub Integration)
- **10-github-init.md (`/github-init`):** Scaffolds GitHub milestones and issues from the task tracker.
- **11-github-create-pr.md (`/github-create-pr`):** Automates PR creation for completed tasks.
- **12-github-sync.md (`/github-sync`):** Full reconciliation between the task tracker and GitHub.

## Usage

This project is intended to be used as a template. To apply this workflow to a new project:

1.  **Copy the Files:** Copy this entire directory structure into the root of your new project.
2.  **Follow the Sequence:** Start with the `/init` phase and proceed through the planning stages (Requirements -> Architecture -> Design -> Integrations) before moving to Test and Implement.
3.  **Adhere to Hard Gates:** Do not proceed to implementation until all planning artifacts exist in a `docs/` folder and have been signed off.
4.  **Use the Commands:** When interacting with an AI agent, refer it to these files for specific instructions on how to handle each phase. For example: "I want to start the requirements phase. Please follow the instructions in `02-requirements.md`."

### Directory Structure of a Project Using This Workflow
```
project-root/
├── WORKFLOW.md
├── .claude/commands/ (or directly in root)
│   ├── 01-init.md
│   ├── ... (all other command files)
│   └── _standards.md
├── docs/
│   ├── project-init.md
│   ├── requirements.md
│   ├── architecture.md
│   ├── design.md
│   ├── tasks.md (Single source of truth for progress)
│   └── integration-registry.md
└── src/ (Project source code)
```
