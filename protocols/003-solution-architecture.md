# Protocol 003 — Solution Architecture Draft

## Status

Active

## Purpose

Define the standard for elaborating a solution architecture from business rules and test specifications. The solution draft bridges the gap between "what must be true" (BRs + test specifications) and "how we build it" (code). It captures technology choices, component design, interfaces, and data flow before implementation begins.

## Position in the Pipeline

```
SOW → Journey → Use Cases → Business Rules → Test Specifications → Solution Draft → Implementation
                                                               ↑
                                                         (this protocol)
```

This protocol governs Stage 6, between Test Specifications and Implementation.

## Rules

### 1. Solution Draft Content

Every solution draft MUST include:

- **Component diagram** — what runs where, how components connect
- **Interface definitions** — data structures, API contracts, message schemas
- **Dependency map** — which libraries each component uses
- **Data flow** — how a request moves through the system end-to-end
- **Error handling** — how each component handles each failure class
- **Build & deploy** — build targets, container images, deployment manifests
- **Cross-component service references** — every external Service consumed across release or namespace boundaries (see Rule 7)
- **Traceability matrix** — which BRs and test specification scenarios each component satisfies

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

### 7. Cross-Component Service References

Every solution document whose scope includes component design MUST contain a "Cross-Component Service References" section. The section enumerates every external Service (or other named resource — Secret, ConfigMap, PVC) that the designed component consumes across Helm release or namespace boundaries.

For each entry, the section MUST specify:

| Field | Description |
|-------|-------------|
| **Consumed resource** | The name and kind of the external Service or resource |
| **Canonical placement** | The namespace where the provider is expected to run, per the project's ADR or architecture document (Protocol 001, Architectural Decision Records) |
| **Helm override value** | The name of the Helm value in the *consuming* chart that exposes the URL (or namespace component) as configurable, so non-default placements can be accommodated at install time |
| **Defaults aligned** | Confirmation that the chart value default matches the canonical FQDN, and that any source-code fallback (hardcoded env-var default) matches the chart value default. If the BR mandates fail-to-start-if-unset, state "N/A — env var required; no source fallback." |

If the component being designed consumes no external Services across release or namespace boundaries, the section MUST be present and explicitly state "None — this component consumes no cross-namespace Services."

A Stage 6 gate cannot PASS unless this section is present and contains entries for every cross-release/cross-namespace Service reference introduced or modified by the solution (see Protocol 001, Stage 6 gate).

**Why this rule exists**: when a solution doc omits cross-component Service addresses, downstream agents (Test Architect, Developer) must infer the URL, the namespace, and the override mechanism from observed code or BRs. Inference is not knowledge — it produces subtle mismatches that only appear at runtime. Requiring the section at Stage 6 surfaces the constraint at design time, when it is cheapest to fix. This rule applies the same "make the implicit explicit" principle as Protocol 001's Architectural Decision Records and Protocol 007's Agent Behavior on Undocumented Architecture.

## Gate

Solution draft reviewed and accepted. All components have defined interfaces. Cross-Component Service References section present and complete (Rule 7). Traceability matrix covers all BRs and test specification scenarios. Ready to implement.
