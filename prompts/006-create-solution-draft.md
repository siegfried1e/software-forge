# Prompt 006 — Create Solution Architecture Draft

## Context

Stage 5.5 of Protocol 001 (Documentation-Driven Development), governed by Protocol 003 (Solution Architecture Draft). This prompt is used after test cases are written, to design the technical solution before implementation.

## Prompt

You are creating a solution architecture draft.

Given the following inputs:

**Business Rules**: [list of BRs, e.g., BR-PROJ-001 through BR-PROJ-010]
**Test Cases**: `test/business-rules/BR-PROJ-*`
**SOW**: `docs/SOW.md`
**Use Cases**: [list of UCs, e.g., UC-PROJ-001 through UC-PROJ-005]

### Technology Stack

| Component | Technology |
|-----------|-----------|
| [Component A] | [language/framework] |
| [Component B] | [language/framework] |
| ... | ... |

### Deliverables

Create the following files in `docs/solution/`:

#### 1. `overview.md` — System Architecture Overview

- Component diagram (ASCII) showing all components and their connections
- Which language each component is written in
- Communication paths between components
- Deployment topology

#### 2. One file per major component

For each component, detail:
- Modules and their responsibilities
- Dependencies (libraries/crates/packages)
- Configuration (environment variables, config files)
- Error handling strategy
- Traceability to BRs

#### 3. `build-deploy.md` — Build and Deployment

- Build targets and commands
- Container images (multi-stage builds)
- Deployment manifests or charts
- CI pipeline
- Development workflow

#### 4. `traceability.md` — BR-to-Component Traceability Matrix

A table mapping every BR to:
- Which component(s) enforce it
- Which module/function implements the enforcement
- Which test file verifies it

```
| BR | Component | Module | Test File |
|----|-----------|--------|-----------|
| BR-PROJ-001 | [component] | [module/file] | BR-PROJ-001_test |
| BR-PROJ-002 | [component] | [module/file] | BR-PROJ-002_test |
| ... | ... | ... | ... |
```

## Rules

- Every component MUST have defined interfaces
- Every design decision MUST trace back to a BR, UC, or protocol
- Error messages in the design MUST match the exact error messages in the test cases
- Use ASCII diagrams, not image references
- Keep each file focused and scannable — solution draft, not a textbook

## Expected Output

Files in `docs/solution/` with all components having defined interfaces, all BRs mapped to components. Ready to implement.
