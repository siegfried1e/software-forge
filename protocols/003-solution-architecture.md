# Protocol 003 — Solution Architecture Draft

## Status

Active

## Purpose

Define the standard for elaborating a solution architecture from business rules and test cases. The solution draft bridges the gap between "what must be true" (BRs + tests) and "how we build it" (code). It captures technology choices, component design, interfaces, and data flow before implementation begins.

## Position in the Pipeline

```
SOW → Journey → Use Cases → Business Rules → Test Cases → Solution Draft → Implementation
                                                               ↑
                                                         (this protocol)
```

This protocol adds Stage 5.5 between Test Cases and Implementation.

## Rules

### 1. Solution Draft Content

Every solution draft MUST include:

- **Component diagram** — what runs where, how components connect
- **Interface definitions** — data structures, API contracts, message schemas
- **Dependency map** — which libraries each component uses
- **Data flow** — how a request moves through the system end-to-end
- **Error handling** — how each component handles each failure class
- **Build & deploy** — build targets, container images, deployment manifests
- **Traceability matrix** — which BRs and test cases each component satisfies

### 2. One Component Per File

Each component gets its own design file in `docs/solution/`:
```
docs/solution/
  overview.md              — system-wide architecture
  component-a.md           — first component
  component-b.md           — second component
  build-deploy.md          — build and deployment
  traceability.md          — BR-to-component mapping
```

### 3. Interface-First Design

Define interfaces (traits, protocols, API contracts) before implementation details. Components communicate through defined contracts, not internal knowledge.

### 4. Traceability

Every design decision MUST trace back to a BR or UC:
- "This validation exists because of BR-PROJ-001"
- "This message schema satisfies UC-PROJ-003"

### 5. Technology Stack

Document the chosen technology stack with rationale for each choice:

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Component A** | [language/framework] | [why] |
| **Component B** | [language/framework] | [why] |
| ... | ... | ... |

### 6. Existing Components Stay

When integrating with an existing system, clearly delineate what is new vs. what already exists. Do not redesign existing components unless necessary.

## Gate

Solution draft reviewed and accepted. All components have defined interfaces. Traceability matrix covers all BRs and test cases. Ready to implement.
