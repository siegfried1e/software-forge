# Protocol 001 — Documentation-Driven Development

## Status

Active

## Purpose

Define the standard methodology for turning a high-level idea into implementation-ready specifications. Every feature or architectural change follows this pipeline before any code is written.

## The Pipeline

```
SOW → User Journey → Use Cases → Business Rules → Test Cases → Solution Draft → Implementation
```

Each stage feeds the next. No stage may be skipped.

### Stage 1: Statement of Work (SOW)

**Input**: A high-level idea, architectural decision, or feature request.

**Action**: Write or revise `docs/SOW.md` to capture:
- What the system is and what it does
- Core concepts and building blocks
- Architectural layers and component roles
- Typical flow (how things work end-to-end)
- Implementation phases with gates
- Key differences from prior versions (if revising)

**Output**: A single document that anyone can read to understand the system.

**Gate**: SOW is reviewed and accepted before proceeding.

### Stage 2: User Journey

**Input**: The SOW.

**Action**: Write user journeys in `docs/user-journeys/` that describe how a human actually uses the system step-by-step. Journeys are concrete, showing:
- Real manifests or configuration
- Real CLI commands
- Real system responses
- Verification steps

**Output**: Numbered journey files (e.g., `01-getting-started.md`).

**Gate**: Journeys cover the primary workflows described in the SOW.

### Stage 3: Use Cases

**Input**: User journeys.

**Action**: Formalize each journey step into use cases in `docs/use-cases/`. Each use case follows the standard template:
- Identifier, Title, Domain, Summary
- Actors, Preconditions
- Main Flow (with actual dialogue for conversational interfaces)
- Alternative Flows
- Error Flows
- Postconditions
- Business Rules Applied (back-referenced after Stage 4)
- References

**Output**: Numbered UC files (e.g., `UC-PROJ-001_UserRegistration.md`).

**Gate**: Every journey step has at least one use case. Every use case has alternative and error flows.

### Stage 4: Business Rules

**Input**: Use cases.

**Action**: Extract enforceable rules from use cases and write them in `docs/business-rules/`. Each rule follows the standard template:
- Identifier, Title, Domain
- Normative Statement (MUST/SHOULD/MAY per RFC 2119)
- Scope (what it covers and does not cover)
- Consequences (what happens on violation)
- Valid and Invalid Examples
- References

After creating rules, back-reference them in the use cases that triggered them.

**Output**: Numbered BR files (e.g., `BR-PROJ-001_PasswordComplexity.md`).

**Gate**: Every use case references at least one business rule. Every business rule is referenced by at least one use case. Cross-references are bidirectional.

### Stage 5: Test Specifications

**Input**: Business rules.

**Action**: For each BR, design a test specification in `docs/test-specifications/` covering:
- One success scenario per valid example
- One failure scenario per invalid example (with exact error message + BR identifier)
- At least one edge case scenario per BR

Test specifications are design documents, not code. They define what to test, not how. See Protocol 002 for full rules.

**Output**: Specification files at `docs/test-specifications/BR-PROJ-NNN_<ShortName>_test-spec.md`.

**Gate**: Every BR has a test specification. Every specification has at least one success, one failure, and one edge case scenario. Every scenario has defined input, expected outcome, and BR reference.

### Stage 6: Solution Architecture Draft

**Input**: Test cases + business rules + technology stack.

**Action**: Design the technical solution in `docs/solution/`. Define components, interfaces, data flow, error handling, build/deploy, and a traceability matrix mapping every BR to its enforcing component. See Protocol 003 for full rules.

**Output**: Files in `docs/solution/`.

**Gate**: All components have defined interfaces. All BRs mapped to components. Solution document includes a "Cross-Component Service References" section per Protocol 003, Rule 7, with entries for every cross-release/cross-namespace Service the designed component consumes. Solution reviewed and accepted.

### Stage 7: Implementation (Red-Green-Refactor)

**Input**: Solution draft + test specifications + use cases + business rules.

**Action**: For each BR in dependency order, implement code using strict Red-Green-Refactor TDD cycles (Protocol 004):
1. **RED** — Read the test specification, write a test, confirm it fails
2. **GREEN** — Write the minimum code to make it pass
3. **REFACTOR** — Clean up while keeping all tests green
4. **Commit** — One commit per color change
5. **Repeat** — Next test in the specification, then next BR

Work through BRs in dependency order (foundational validation first, integration last). One test at a time. No code without a failing test. No refactoring while red.

**Output**: Test code and working implementation in the packages defined by the solution draft.

**Gate**: All tests pass (green). Every test traces back to a test specification. Implementation matches the solution draft interfaces and the verification steps in the use cases. Every commit follows the RED-GREEN-REFACTOR pattern. Before each dispatch, the Orchestrator has run the Pre-Flight Architectural-Pivot Checkpoint (see below) and resolved any signals that fired.

#### Stage 7 Pre-Flight Architectural-Pivot Checkpoint

Before any Stage 7 Developer dispatch ships, the Orchestrator MUST evaluate all five questions and confirm that each answer is either "No" or "Yes — already covered upstream by [specific BR / Solution / ADR citation]". If any answer is "Yes" without upstream coverage, the Orchestrator MUST halt the Stage 7 dispatch and dispatch the missing upstream stage(s) first.

1. **New K8s (or platform) resource kind**: does this dispatch's acceptance criteria CREATE a kind of resource that did not exist in the project's deployment surface before — Secret, ConfigMap, Role, ClusterRole, NetworkPolicy, Job, CronJob, ServiceAccount, CRD, Webhook, mutating/validating admission policy, custom resource? If yes → architectural; require Stage 4 BR + Stage 6 Solution + ADR before Stage 7.

2. **Storage / state substrate change**: does this dispatch CHANGE the substrate of an existing data flow — hostPath ↔ PVC, hostPath ↔ Secret/ConfigMap, in-memory ↔ persistent, single-process ↔ multi-process, single-cluster ↔ external managed service, sync ↔ async/queued? Substrate changes alter trust boundary, multi-node story, durability story, and management UX. If yes → architectural; require ADR + BR + Solution.

3. **New contract surface**: does this dispatch INTRODUCE a new HTTP route, gRPC method, NATS / Kafka / RabbitMQ subject, event topic, webhook callback, or any other inter-process contract surface? If yes → require Contract Registry entry (Solution Architect dispatch) before Stage 7.

4. **New RBAC binding or cross-namespace access**: does this dispatch ADD a new ServiceAccount, Role/ClusterRole, RoleBinding/ClusterRoleBinding, cross-namespace Service reference, or otherwise expand the principal-to-resource access matrix? If yes → require Stage 6 Cross-Component Service References update (Protocol 003, Rule 7) + Solution Architect dispatch.

5. **Authorization-citation mismatch**: does this dispatch CITE an ADR, BR, Solution doc, or prior dispatch as authorization for its design? If yes, the Orchestrator MUST read the cited section verbatim and confirm the dispatch design matches. Citation-mismatch (the cited document explicitly says X, the dispatch ships not-X or beyond-X) is a stop-and-question signal regardless of how clean the rest of the dispatch looks. Do not dispatch until the contradiction is resolved by either an upstream amendment (with proper stage gate) or a redesign of the dispatch to match the cited authorization.

The Stage 7 gate CANNOT mark PASS if any of these signals fired during dispatch authoring and was not resolved upstream first. The Orchestrator records the checkpoint evaluation (per dispatch) in the audit-log gate verdict, even when all answers are "No" — the negative confirmation is itself the evidence that the check was run.

**Scope clarifications**:

- This rule does NOT slow down genuine tactical fixes. A 1-line value-default change, a typo correction, a comment update, a test added for an existing scenario — none of these trigger any of the 5 questions. The checkpoint is a 60-second pass for clean dispatches; only architectural drift triggers the full Stage 1-6 redirect.
- This rule does NOT require pre-dispatch consensus on every chart tweak or code change. Solo Developers can self-evaluate the 5 questions and proceed for clean cases; the Orchestrator only formally invokes the gate when authoring multi-agent dispatches.
- This rule does NOT retroactively undo merged work. When detection happens after Stage 7 merge, remediation = retroactive Stage 1-6 formalization, NOT code rollback. The "working software is good" guidance stays.
- Each "Yes" answer that requires upstream coverage SHOULD cite the specific upstream artifact (BR-N, Solution §N, ADR #N) that authorizes the design. "Yes — covered by Solution §3.2 + BR-145" is acceptable; "Yes — Solution covers it" is not specific enough and shifts gate burden to readers.

**Why this rule exists**: a Stage 7 dispatch framed as a "tactical fix" can silently introduce architectural surface area — a new Kubernetes resource kind, a substrate change, a new inter-process contract, a new RBAC binding — while citing upstream authorization that does not actually cover the design. The cost of catching this after Stage 7 merge is a multi-dispatch retroactive formalization cascade (Analyst → Specifier → Test Architect → Solution Architect → Developer revalidation) for every undocumented decision that shipped. The cost of catching it at pre-flight is 60 seconds per dispatch. This rule applies the same "make the implicit explicit" principle as Protocol 001's Architectural Decision Records, Protocol 003's Cross-Component Service References, and Protocol 002's Runtime Behavior Coverage Check.

## Architectural Decision Records

Any decision that shapes the project's architecture beyond what is captured by an SOW or a business rule MUST be recorded in an Architectural Decision Record (ADR) before it is encoded in code, charts, BRs, or solution docs.

### What Requires an ADR

- Component-level placement choices (which namespace, which directory, which module)
- Cross-component ownership decisions (which agent or chart owns a given resource)
- Technology selection or substitution (language, framework, library, infrastructure)
- Deployment topology (single vs. multiple replicas, queue group vs. single consumer, etc.)
- Pattern adoption not from a BR (e.g., "we use control-plane/data-plane namespace split")
- Cross-cutting conventions (naming schemes, error formats, log structure)

### Location

ADRs MAY live in `docs/coordination/decision-log.md` (project-side) or in a dedicated `docs/architecture/` directory. The location is project-specific; the requirement that the decision be recorded is universal. A canonical ADR template is provided in `templates/decision-record.md`.

### Timing

ADRs MUST be written BEFORE the decision is encoded in code, charts, BRs, or solution docs. When a decision was already applied without documentation, it MUST be recorded retroactively before the next phase's Stage 7 begins. The retroactive path (backfill dispatch or bundle into next phase) is defined by the Retroactive Formalization rule — see Protocol 006, Rule 9, and Prompt 016.

### ADRs and the Pipeline

An architectural decision that constrains a BR MUST be referenced in that BR's normative statement. A decision that constrains the solution MUST be referenced in the solution doc's component definitions. The Orchestrator MAY dispatch the Solution Architect or Specifier to write an ADR if a decision surfaces during Stage 6 or Stage 4.

Agents encountering architectural choices that cannot be traced to an authoritative document MUST stop and flag the gap rather than infer or invent a rationale. See Protocol 007 (Agent Behavior on Undocumented Architecture).

## Rules

1. **No code without documentation.** Every implementation MUST trace back to a use case and business rule.
2. **Stages are sequential.** You MUST complete each stage's gate before starting the next.
3. **Changes flow forward.** If the SOW changes, user journeys, use cases, and business rules MUST be reviewed and updated.
4. **Cross-references are mandatory.** Use cases reference business rules. Business rules reference use cases. Both reference protocols.
5. **Prompts drive execution.** Each stage transition SHOULD be captured as a prompt in `prompts/` so the work is reproducible and auditable.

## Prompt Commands

Each stage has a corresponding prompt in `prompts/` that can be used to execute it:

| Stage | Prompt | Purpose |
|-------|--------|---------|
| 1. SOW | `prompts/001-create-sow.md` | Create or revise the Statement of Work |
| 2. Journey | `prompts/002-create-user-journey.md` | Create a user journey from the SOW |
| 3. Use Cases | `prompts/003-create-use-cases.md` | Formalize journey steps into use cases |
| 4. Business Rules | `prompts/004-create-business-rules.md` | Extract rules from use cases and cross-reference |
| 5. Test Specifications | `prompts/005-create-br-test-cases.md` | Design test specifications from business rules |
| 6. Solution Draft | `prompts/006-create-solution-draft.md` | Design technical solution with interfaces and traceability |
| 7. Implementation | `prompts/007-implement-red-green-refactor.md` | Implement code via strict Red-Green-Refactor TDD cycles (Protocol 004) |

These prompts are reusable. Each time a new feature is started, run them in sequence.
