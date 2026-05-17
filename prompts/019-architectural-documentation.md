# Prompt 019 — Architectural Decisions MUST Be Documented; Agents Question, Never Invent

> **Source**: drafted by Mila Orchestrator (`/home/themaster/workspace/agentic`) on 2026-05-16 after the trigger described below.
> **To submit**: copy this file to `/home/themaster/workspace/evocatio/software-forge/prompts/019-architectural-documentation.md`, review/refine, and integrate into the relevant protocol(s).

## Context

While Mila was closing Phase 6 (Agent Messaging via NATS), the user asked the Orchestrator a simple question: "why are there two namespaces, `mila` and `mila-system`?". The Orchestrator answered with a plausible-sounding rationale — control plane vs data plane, multi-tenant readiness, Kubernetes convention. The answer was internally consistent and matched how similar platforms (cert-manager, ArgoCD, Istio) organize themselves.

**But the rationale was nowhere documented.** No decision-log entry, no SOW section, no BR. The pattern was load-bearing — referenced by every chart's `.Release.Namespace`, by BR-MILA-144 (Secret in `mila`), by the operator's RBAC scope, by NATS DNS, by every Phase 6 deployment. Yet the *why* was never recorded. The Orchestrator was **inferring** the rationale post-hoc and presenting it as known design intent.

This is a methodology failure mode. When architectural decisions are implicit:

1. **Downstream agents inherit the gap.** Specifier writing BRs against the implicit pattern, Test Architect writing scenarios, Solution Architect designing flows — each has to *infer* the constraint from observed code/charts. Inference is not knowledge.
2. **Inferred rationales drift.** A decision documented once stays consistent across decades. A decision inferred separately by ten agents produces ten slightly-different rationales, and the ones that propagate downstream may not match the original intent.
3. **Friction surfaces as bugs.** Mila's BR-MILA-144 says the `mila-nats-creds` Secret MUST be in the `mila` namespace, but the post-Phase-6 `mila-agent-worker` chart templates it in `.Release.Namespace` (= `mila-system`). The BR was written assuming namespace placement that the chart's design contradicts. Neither side documented the namespace decision, so neither side caught the contradiction until integration testing surfaced it as a discrepancy.

The user's response when the Orchestrator described this issue:

> "toute décision qui affecte l'architecture doit être documenté, à s'assurer d'avoir dans le software-forge comme une règle importante, et que l'agent n'invente pas, qu'il questionne."

(Every decision that affects architecture must be documented, make sure it's in software-forge as an important rule, and that the agent doesn't invent — it questions.)

Two intertwined rules:

- **(R1) Documentation precedes application.** Any architectural decision MUST be recorded (in a decision-log, a solution doc, or another authoritative location) BEFORE it is encoded in code, charts, BRs, tests, or solution drafts.
- **(R2) Agents question, never invent.** When an agent of any role encounters an architectural choice not traceable to a documented source, the agent MUST stop and either ask the user OR (if the role permits) write a retroactive entry to document the decision before proceeding. The agent MUST NOT fabricate rationale to fill the gap.

## Prompt

Add these two rules to the software-forge methodology. Concrete proposals:

### Part A — Protocol 001 (Documentation-Driven Development): add "Architectural Decision Records"

Protocol 001 already defines the documentation cascade (SOW → user journey → use cases → business rules → test specifications → solution draft → implementation). Add a new section after the stage definitions:

> **Architectural Decision Records (ADRs)**
>
> Any decision that shapes the project's architecture beyond what is captured by an SOW or a business rule MUST be recorded in an Architectural Decision Record. Examples of decisions that require an ADR:
>
> - Component-level placement choices (which namespace, which directory, which module)
> - Cross-component ownership decisions (which agent or chart owns a given resource)
> - Technology selection or substitution (language, framework, library, infrastructure)
> - Deployment topology (single vs multiple replicas, queue group vs single consumer, etc.)
> - Pattern adoption that's not from a BR (e.g., "we use control-plane/data-plane namespace split")
> - Cross-cutting conventions (naming schemes, error formats, log structure)
>
> ADRs MAY live in `docs/coordination/decision-log.md` (project-side) or in a dedicated `docs/architecture/` directory if the project chooses. The location is project-specific; the requirement that the decision be recorded is universal. ADRs MUST be written BEFORE the decision is encoded in code, charts, BRs, or solution docs — except when the decision is being recorded retroactively (see Protocol 016's Retroactive Formalization pattern, which applies to ADRs the same way it applies to ad-hoc fixes).

### Part B — Protocol 007 (Multi-Agent Pipeline Execution): add "Agent Behavior on Undocumented Architecture"

Protocol 007 defines per-role responsibilities and the Orchestrator's prohibitions. Add a subsection (likely under Operating Principles or as a new top-level section parallel to "Developer Agent Prohibitions"):

> **Agent Behavior on Undocumented Architecture**
>
> When ANY agent — Orchestrator, Analyst, Specifier, Test Architect, Solution Architect, or Developer — encounters an architectural choice that is referenced by upstream artifacts (code, charts, BRs, etc.) but cannot be traced to an authoritative document (ADR, decision-log, SOW section, solution doc, or BR), the agent MUST:
>
> 1. **Stop**. Do not assume the rationale. Do not invent a justification "that sounds right".
> 2. **Flag the gap**. In the completion report, list the undocumented decision under a `Discovered Architectural Gaps` section.
> 3. **Either** (a) ask the user (if the agent has a synchronous channel — e.g., the Orchestrator can ask in the chat), or (b) report `blocked` (if the agent operates asynchronously and the gap blocks their deliverables), or (c) proceed with explicit assumption marked as "ASSUMPTION (pending ADR): ..." if and only if the dispatch's acceptance criteria allow it and the assumption is clearly labeled as such.
>
> The phrase "I think the reason is..." or "this is probably because..." in an agent's reasoning is a signal that the agent is inventing. The correct response to that signal is to stop and apply step 1.
>
> The Orchestrator's role-specific responsibility: when the Orchestrator discovers an undocumented decision during gate evaluation, completion-report review, or any other activity, the Orchestrator MUST either write a retroactive ADR (within Orchestrator write scope — `docs/coordination/`) or ask the user for the rationale. Dispatching downstream agents against undocumented architecture is forbidden.

### Part C — README.md table update

Update the protocols and prompts tables in `software-forge/README.md` to reference the new ADR section and the new behavioral rule.

## Expected Output

1. **Protocol 001** — new "Architectural Decision Records" section (~25-40 lines) added near the stage definitions; cross-references Protocol 016 (Retroactive Formalization).
2. **Protocol 007** — new "Agent Behavior on Undocumented Architecture" subsection (~20-30 lines); placed near the existing Prohibitions subsections (Orchestrator Prohibitions, Developer Agent Prohibitions).
3. **README.md** — both protocols' entries updated to mention the new content.
4. **Optional: a new template file** — `templates/decision-record.md` providing the canonical ADR format (id, status, context, decision, rationale, consequences, cross-references) for projects that adopt the formal ADR location.

## Read Before Editing

- `protocols/001-documentation-driven-development.md` — stage definitions; the natural insertion point for ADRs.
- `protocols/007-multi-agent-pipeline-execution.md` — agent role definitions; the natural insertion point for the behavioral rule.
- `protocols/016-retroactive-formalization.md` (if exists at integration time — based on Prompt 016) — cross-reference; retroactive ADRs are an application of the retroactive-formalization pattern.
- Prompts 013 (Developer Agent Prohibitions), 015 (Orchestrator Prohibitions) — formatting precedent for behavioral rules.

## Triggering Incident (for evidence in the prompt's context section)

- **Date**: 2026-05-16, end of Mila Phase 6 closure (after D-091 PASS).
- **Decision**: Mila's `mila` (data plane) vs `mila-system` (control plane) namespace split.
- **Implicit since**: at latest Phase 1 / Phase 2 (operator deployed in `mila-system`). Probably earlier.
- **Documented in**: nothing, until Mila decision-log #017 (this prompt's project-side counterpart) was added retroactively on 2026-05-16.
- **Friction observed**: BR-MILA-144 says Secret in `mila`, post-D-088 chart templates in `mila-system`. The mismatch surfaced during D-089 integration gate; documented as a "potential FD" but never blocked the gate. Without this rule, the gap could have persisted indefinitely.
