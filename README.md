# Software Forge

A documentation-driven development methodology that turns ideas into working software through a structured, repeatable pipeline.

## The Pipeline

```
SOW → User Journey → Use Cases → Business Rules → Test Cases → Solution Draft → Implementation
 │         │              │             │              │              │              │
 │         │              │             │              │              │              └─ Protocol 004
 │         │              │             │              │              └─ Protocol 003
 │         │              │             │              └─ Protocol 002
 │         │              │             │
 └─────────┴──────────────┴─────────────┴──────────────── Protocol 001 (stages 1-4)
```

Every feature follows this pipeline. No stage may be skipped. Each stage feeds the next.

## Protocols

Protocols define **how** to do each thing:

| # | Protocol | Governs |
|---|----------|---------|
| 001 | [Documentation-Driven Development](protocols/001-documentation-driven-development.md) | The full pipeline — stages 1 through 7; Architectural Decision Records (ADRs) |
| 002 | [Test-Driven Business Rules](protocols/002-test-driven-business-rules.md) | Stage 5 — writing test cases from BRs; runtime behavior coverage check (E2E scenarios) |
| 003 | [Solution Architecture](protocols/003-solution-architecture.md) | Stage 6 — designing the technical solution; cross-component service references |
| 004 | [Red-Green-Refactor](protocols/004-red-green-refactor.md) | Stage 7 — implementing with strict TDD |
| 005 | [Cross-Project Integration](protocols/005-cross-project-integration.md) | Multi-project boundaries — contracts between systems |
| 006 | [Phased Multi-Component Delivery](protocols/006-phased-multi-component-delivery.md) | Multi-component systems — master SOW, build order, vertical slices |
| 007 | [Multi-Agent Pipeline Execution](protocols/007-multi-agent-pipeline-execution.md) | Multi-agent decomposition — agent roles, orchestration, context handoff, workspace isolation, developer agent prohibitions, orchestrator prohibitions, agent behavior on undocumented architecture |
| 008 | [Agent Communication Contract](protocols/008-agent-communication-contract.md) | Dispatch, completion, and gate feedback artifacts between Orchestrator and agents |

## Prompts

Prompts are **executable instructions** — give them to an AI assistant or follow them yourself:

| # | Prompt | Pipeline Stage |
|---|--------|---------------|
| 001 | [Create SOW](prompts/001-create-sow.md) | Stage 1 |
| 002 | [Create User Journey](prompts/002-create-user-journey.md) | Stage 2 |
| 003 | [Create Use Cases](prompts/003-create-use-cases.md) | Stage 3 |
| 004 | [Create Business Rules](prompts/004-create-business-rules.md) | Stage 4 |
| 005 | [Create BR Test Specifications](prompts/005-create-br-test-cases.md) | Stage 5 |
| 006 | [Create Solution Draft](prompts/006-create-solution-draft.md) | Stage 6 |
| 007 | [Implement (Red-Green-Refactor)](prompts/007-implement-red-green-refactor.md) | Stage 7 |
| 008 | [Define Agent Communication Contract](prompts/008-define-agent-communication-contract.md) | Protocol 008 |
| 009 | [Clarify Agent Identity vs Instance](prompts/009-clarify-agent-identity-vs-instance.md) | Protocol 007 |
| 010 | [Orchestrate Pipeline](prompts/010-orchestrate-pipeline.md) | Orchestrator — triage, routing, propagation |
| 011 | [Recurring Discrepancy Rule](prompts/011-recurring-discrepancy-rule.md) | Protocol 007 — recurring discrepancies in triage |
| 012 | [Parallel Workspace Isolation](prompts/012-parallel-workspace-isolation.md) | Protocol 007 — workspace isolation for parallel agents |
| 013 | [Developer Agent Prohibitions](prompts/013-developer-agent-prohibitions.md) | Protocol 007 — behavioral constraints for Stage 7 developer agents |
| 014 | [Dispatch Deliverable Grouping](prompts/014-dispatch-deliverable-grouping.md) | Protocol 008 — deliverable grouping for high-count dispatches |
| 015 | [Orchestrator Prohibitions](prompts/015-orchestrator-prohibitions.md) | Protocol 007 — orchestrator write-scope prohibitions |
| 016 | [Retroactive Formalization](prompts/016-retroactive-formalization.md) | Protocol 006 — formalization debt tracking and resolution |
| 017 | [Dispatch Context Reset](prompts/017-dispatch-context-reset.md) | Protocol 008 — context reset field for fresh agent sessions |
| 018 | [Self-Contained Dispatch](prompts/018-self-contained-dispatch.md) | Protocol 008 — self-containment requirement for reset dispatches |
| 019 | [Architectural Documentation](prompts/019-architectural-documentation.md) | ADR requirement (Protocol 001) and agent behavior on undocumented architecture (Protocol 007) |
| 020 | [Stage 6 Gate — Cross-Component Service References](prompts/020-stage6-gate-cross-component-services.md) | Protocol 003 — mandatory enumeration of external services consumed across release/namespace boundaries |
| 021 | [Stage 5 Gate — Runtime E2E Tests](prompts/021-stage5-gate-runtime-e2e-tests.md) | Protocol 002 — runtime behavior coverage check (E2E scenarios required for runtime BRs) |

## Templates

Reusable configuration files for projects adopting this methodology:

| Template | Purpose |
|----------|---------|
| [Developer Agent CLAUDE.md](templates/developer-claude-md.md) | CLAUDE.md template for Stage 7 developer agents — enforces behavioral prohibitions (no writes to coordination directory, no branch merging, no self-issued gate verdicts) |
| [Architectural Decision Record](templates/decision-record.md) | Canonical ADR template — use for documenting architectural decisions before encoding them in code, charts, BRs, or solution docs |
| [Test Specification with E2E Scenario](templates/test-spec-with-e2e.md) | Test specification template including the canonical end-to-end scenario shape for BRs that specify runtime behavior (Protocol 002, Rule 8) |

## How to Use

1. **Start a new feature**: Run Prompt 001 to create a Statement of Work
2. **Walk through the pipeline**: Run each subsequent prompt in order (002 → 007)
3. **Gate each stage**: Don't proceed until the current stage's gate conditions are met
4. **Adapt the templates**: Replace `PROJ` with your project identifier (e.g., `BR-MYAPP-001`)

The methodology is language-agnostic and framework-agnostic. The examples use generic placeholders — adapt the test file extensions, build commands, and tooling to your stack.

## Key Principles

- **No code without documentation** — every line traces back to a use case and business rule
- **Tests are the specification** — written before implementation, in a failing state
- **One test at a time** — strict Red-Green-Refactor discipline
- **Cross-references are mandatory** — use cases reference business rules and vice versa
- **Prompts drive execution** — reproducible, auditable stage transitions

## License

MIT
