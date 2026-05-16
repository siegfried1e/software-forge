# Protocol 006 — Phased Multi-Component Delivery

## Status

Active

## Purpose

Define how to plan and execute a system built from multiple components, where each component is delivered incrementally through its own Protocol 001 pipeline run. This protocol governs the layer above individual pipeline runs — the build order, the master plan, and the coordination between parallel workstreams within each phase.

## The Problem

Protocol 001 assumes a single project running a single pipeline. Real systems often consist of multiple projects or components that must be built in a deliberate order, where later components depend on earlier ones. Without a governing structure:

- Components get built in the wrong order, creating integration pain
- Shared concerns (UI, contracts, observability) are bolted on after the fact instead of built alongside
- There is no single document that explains what gets built when and why

## Concepts

### Master SOW

A top-level Statement of Work that sits above individual project SOWs. It defines the system as a whole, the component inventory, the build order, and the phased delivery plan. It does NOT replace per-component SOWs — it frames them.

### Phase

A phase delivers one vertical slice of the system. A phase typically adds one component (or a small cohesive group) and runs Protocol 001 for each affected project. A phase is complete when all its pipeline runs reach their gates.

### Vertical Slice

A phase delivers end-to-end functionality, not just backend or just frontend. If a component includes a user-facing surface (dashboard, CLI, API), that surface is part of the same phase — not deferred to a later one.

## Rules

### 1. Master SOW Before Component SOWs

Before running Protocol 001 for any component, a master SOW MUST exist. It covers:

- **System overview** — what the system does, in plain language
- **Component inventory** — every component, its role, and which project owns it
- **Build order** — the sequence of phases, with rationale for the ordering
- **Phase definitions** — for each phase: what components are delivered, what pipeline runs are triggered, what the phase gate is
- **Cross-cutting concerns** — shared surfaces (dashboard, CLI, gateway) that grow with each phase
- **Out of scope** — what is explicitly deferred beyond the last defined phase

### 2. Master SOW Location

The master SOW lives above individual projects, at the root of the repository or workspace:

```
<workspace>/docs/SOW.md
```

Individual project SOWs remain in their own project directories.

### 3. One Phase at a Time

Phases are executed sequentially. A phase MUST reach its gate before the next phase begins. Within a phase, multiple pipeline runs MAY proceed in parallel if they have no dependencies on each other's outputs.

### 4. Each Phase Triggers Protocol 001

Each component touched in a phase gets its own Protocol 001 run (Stages 1-7). If a phase adds a backend component and a corresponding UI surface, that is two pipeline runs within the same phase.

### 5. Cross-Project Contracts Per Phase

When a phase involves multiple projects, Protocol 005 (Cross-Project Integration) applies. Contracts MUST be defined before implementation begins within the phase. Every contract created or modified during a phase MUST be recorded in the Contract Registry (`docs/contracts/registry.md`) with consumer, provider, version, and the phase that introduced or modified it. The Solution Architect owns this registry.

### 6. Shared Surfaces Grow Incrementally

If the system includes a shared surface (dashboard, CLI, SDK) that accumulates functionality across phases, each phase adds to it — not replaces it. The shared surface has its own pipeline run per phase, scoped to only the new functionality being added.

### 7. Phase Gate

A phase is complete when:

- All Protocol 001 pipeline runs within the phase have passed their Stage 7 gate
- All cross-project contracts (Protocol 005) are satisfied and the Contract Registry (`docs/contracts/registry.md`) is up to date
- The shared surface (if any) includes the new component's functionality
- Integration between the new component and existing components is verified

### 8. Master SOW Evolves

The master SOW is a living document. After each phase, it SHOULD be updated to reflect:

- What was actually delivered vs. planned
- Changes to future phase definitions based on what was learned
- New components discovered during the phase

### 9. Retroactive Formalization

During integration testing (Stage 7) or post-gate deployment, ad-hoc fixes are sometimes applied that bypass the Protocol 001 pipeline — a `kubectl patch`, a direct file edit, a config tweak under time pressure. These fixes are often correct and necessary. They are not inherently prohibited. But every such fix incurs **formalization debt**: behavior exists in the system that no BR describes, no test specification covers, and no test verifies. If the component is regenerated from specifications, the fix disappears.

**The obligation**: each ad-hoc fix incurs formalization debt that MUST be resolved before the next phase's Stage 7 begins. The Orchestrator MUST track each such fix in the coordination records (decision log or a dedicated `docs/coordination/formalization-debt.md`) with:

- What was changed and in which file or resource
- Who applied the fix and when
- Why the pipeline was bypassed (unblocking a gate, time pressure, operational necessity)
- Formalization status: `pending` or `complete`

**Two resolution paths**:

| Path | When to use | What happens |
|------|-------------|-------------|
| **Backfill dispatch** | Fix is isolated; debt should be cleared immediately | Orchestrator dispatches Specifier (BR), Test Architect (test spec), and Developer (test + any code alignment) for the ad-hoc change |
| **Bundle into next phase** | Multiple small fixes accumulate; individual dispatches would be overhead | Fixes are listed as "Phase N stabilization" items in the next phase's Stage 1 SOW and covered alongside new phase work |

**Deadline**: formalization debt from Phase N MUST be resolved before Phase N+1 Stage 7 begins. It MAY be resolved earlier. Debt MUST NOT be carried across two phase boundaries — debt from Phase N cannot remain open when Phase N+2 starts.

**Prevention**: the Orchestrator SHOULD prefer dispatching agents for fixes over applying them directly (see Protocol 007, Orchestrator Prohibitions). The retroactive path exists as a safety net for cases where bypassing the pipeline was genuinely necessary — not as an alternative workflow.

**Cross-reference**: the Orchestrator's responsibility to track and resolve formalization debt is listed in Protocol 007, Orchestrator Responsibilities, item 8.

## Master SOW Template

```markdown
# Master SOW — <System Name>

## Overview
What the system does, who it serves, why it exists.

## Component Inventory

| Component | Role | Project | Phase |
|-----------|------|---------|-------|
| ... | ... | ... | ... |

## Contracts
See [Contract Registry](../contracts/registry.md) (owned by Solution Architect).

## Cross-Cutting Surfaces

| Surface | Type | Grows With |
|---------|------|------------|
| ... | dashboard / CLI / SDK | Each phase adds its component |

## Phases

### Phase 1: <Component Name>

**Delivers**: <what>
**Pipeline runs**:
- `<project-a>`: Protocol 001 for <scope>
- `<project-b>`: Protocol 001 for <scope>
**Contracts**: <project-a> ↔ <project-b> (Protocol 005)
**Gate**: <verifiable condition>

### Phase 2: <Component Name>

...

## Out of Scope
What is explicitly deferred.
```

## Example

A system with a backend API and a dashboard:

```
Phase 1: Products
  ├── backend: Protocol 001 for product catalog API
  ├── dashboard: Protocol 001 for product management UI
  └── contract: dashboard ↔ backend API (Protocol 005)

Phase 2: Orders
  ├── backend: Protocol 001 for order processing API
  ├── dashboard: Protocol 001 for order management UI (added to existing dashboard)
  └── contract: dashboard ↔ backend API updated (Protocol 005)
```

Each phase leaves the system fully functional with everything built so far.

## Anti-Patterns

| Anti-Pattern | Why It's Wrong | Correct Approach |
|-------------|----------------|------------------|
| Build all backends, then all UIs | Defers integration risk; UI assumptions go untested | Vertical slices — each phase delivers end-to-end |
| Skip the master SOW | No shared understanding of build order or dependencies | Master SOW first, then component SOWs |
| Run phases in parallel | Later phases depend on earlier ones; integration breaks | One phase at a time, parallel runs only within a phase |
| Rewrite shared surface each phase | Wasteful; breaks existing functionality | Incremental addition only |
| Freeze the master SOW | Phases reveal new information; rigidity causes drift | Update after each phase |
| Ad-hoc fix with no formalization tracking | Behavior exists in the system with no BR, no test, no audit trail — regenerating from specs removes the fix | Record as formalization debt immediately; resolve before next phase Stage 7 |
| Carry formalization debt across two phase boundaries | Debt compounds; the undocumented behavior becomes load-bearing and harder to backfill | Resolve by Phase N+1 Stage 7 at the latest |
