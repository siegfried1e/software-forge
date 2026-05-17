# Prompt 021 — Stage 5 gate: test specs for BRs covering runtime behavior MUST include end-to-end integration scenarios

> **Source**: drafted by Mila Orchestrator (`/home/themaster/workspace/agentic`) on 2026-05-17 after ADR #037 (Auth Broker vision) surfaced a recurring detection-gap pattern.
> **To submit**: copy this file to `/home/themaster/workspace/evocatio/software-forge/prompts/021-stage5-gate-runtime-e2e-tests.md`, review/refine, and integrate into Protocol 002 (Test-Driven Business Rules) Stage 5 gate criteria.

## Context

The Mila project hit four "detection gap" frictions in Phase 6 closure, each surfacing only at human-manual integration test time despite all upstream gates passing:

| FD | Description | When caught |
|----|-------------|------------|
| **FD-009** | NatsTaskClaimer race condition (worker returned Task to execution loop before operator populated `status.agentSnapshot`) | D-089 Integration manual gate run |
| **FD-010** | `mila-nats-creds` Secret namespace mismatch (BR vs chart) | D-089 deploy testing |
| **FD-011** | `mila-entry` URL hardcoded wrong namespace in dashboard BFF | Post-Phase-6 manual dashboard test |
| **FD-012** | `claudeAuth` chart mounts directory but missing sibling `~/.claude.json` config file | Post-stab3 manual provider test |

All four share a structural cause distinct from the cross-namespace addressing pattern (which is the subject of Prompt 020):

**Each affected BR explicitly or implicitly scoped OUT the runtime behavior of the component it specified**, deferring "actually running this" to integration testing. Each test spec for those BRs covered unit-level scenarios (parsing, validation, chart rendering, label presence) but not end-to-end runtime execution. The Stage 5 gate accepted these scoped-out specs because no protocol criterion required runtime coverage.

Concrete example (worst offender): **BR-MILA-170** ("ClaudeCodeProviderInDashboard") explicitly stated in its Scope section:

> "Does not cover: Runtime behavior differences between `claude-code` and `anthropic` providers (agent-runtime scope)"

This punted to "agent-runtime scope" — but no agent-runtime BR ever picked up the claude-code provider runtime testing either. The runtime path (`Agent CR with provider=claude-code` → operator populates snapshot → worker spawns `claude` subprocess → completion message published) was never end-to-end tested. FD-012 surfaced it at manual test time after Mila was already deep in Phase 6 stab3.

The cost of detection-at-integration vs detection-at-Stage-5:
- **Stage 5**: a test scenario noting "if `claude` binary cannot find `~/.claude.json` config, this must fail with a specific error" would have surfaced the missing chart mount during initial design review, not after multiple stab rounds.
- **Integration**: each FD requires a stab sweep (Specifier amendment + Test Architect amendment + Solution Architect amendment + N Developer dispatches + Integration revalidate). Mila ran one such sweep per FD in Phase 6 closure. Significant cumulative cost.

## Prompt

Add the following gate criterion to Protocol 002 (Test-Driven Business Rules), Stage 5 gate evaluation:

> ### Runtime Behavior Coverage Check (added 202X)
>
> Every BR whose normative statement specifies *runtime behavior of a component* — provider implementations, message claimers/publishers, reconcilers, controllers, subprocess invocation, network calls to external services — MUST have an accompanying test specification that includes at least one **end-to-end integration scenario** that exercises the actual runtime path described.
>
> An end-to-end scenario specification MUST:
> 1. Identify the runtime entry point (e.g., "operator reconcile loop is invoked", "worker claims a NATS message", "BFF receives an HTTP POST").
> 2. Identify the runtime exit point (e.g., "Task CR reaches terminal phase", "completion message published to NATS", "client receives HTTP response with body").
> 3. Specify the cross-component path between entry and exit (e.g., "operator → Task CR → worker → provider → external API → response → completion publish").
> 4. Specify the expected observable signal (CRD field value, NATS message subject + payload, HTTP response code + body, log line, metric increment).
> 5. NOT delegate the runtime path to "<other component>'s scope" — if the BR specifies that behavior X happens, the test spec for that BR must include a scenario that verifies X actually happens end-to-end in the runtime path the BR describes.
>
> The Stage 5 gate CANNOT mark PASS if the BR specifies runtime behavior AND the test spec does not contain at least one such scenario.
>
> **Scope clarifications**:
>
> - This rule does NOT require test specs to dictate implementation language or test framework. The scenario specification describes *what to observe*, not *how to execute*.
> - This rule does NOT apply to BRs that purely specify static contracts (CRD schema, configuration values, naming conventions). Those legitimately have no runtime path.
> - If a BR's runtime path crosses multiple components and the BR's owning agent cannot itself execute the path (e.g., a BR owned by Specifier specifies behavior of code owned by Runtime Developer), the test spec MUST still include the end-to-end scenario; the Test Architect drafts what to observe, the Developer at Stage 7 chooses an implementation (mocked component boundaries with real runtime under test, real cluster integration test, etc.).
> - Test specs MAY mark end-to-end scenarios with explicit `[INTEGRATION]` or `[E2E]` tags to distinguish them from unit-level scenarios. Implementations at Stage 7 MAY satisfy these scenarios via integration test frameworks rather than unit tests, depending on the project's testing infrastructure.

**Reject pattern (must be flagged as violation)**:

Test spec includes ONLY scenarios that mock the underlying runtime (subprocess mocks, HTTP mocks, fake CRD stores) when the BR specifies behavior that depends on real component interaction.

**Accept pattern (compliant)**:

Test spec includes scenarios covering:
- Unit-level (mocked dependencies, fast feedback)
- AND at least one end-to-end scenario (real cross-component path, observable runtime signal)

## Cross-references in Mila context

This prompt is informed by:

- **ADR #037** (Mila Auth Broker vision, 2026-05-17): the methodology improvement clause explicitly proposed amending Stage 5 gate to mandate E2E provider tests, motivated by the FD-009/010/011/012 cluster of detection-gap incidents.
- **FD-012** (`claudeAuth` missing sibling file mount): the most direct example — BR-MILA-170 scoped out runtime, no test exercised the claude subprocess path, the mount gap surfaced only at manual test time months after BR-170 was specified.
- **Prompt 019** (Architectural Decisions MUST Be Documented): related methodology rule. Prompt 019 prevents undocumented architecture; Prompt 021 prevents un-runtime-tested architecture. Same family of "make the implicit explicit" rules.
- **Prompt 020** (Stage 6 gate Cross-Component Service References): the Stage 6 sibling rule for cross-namespace addressing.

The three prompts together (019, 020, 021) form a coherent methodology hardening pass:
- 019: explicit decisions at all stages
- 020: explicit Service references at Stage 6
- 021: explicit runtime coverage at Stage 5

Each addresses a distinct class of detection gap Mila hit in Phase 6 closure.

## Expected Output

1. **Protocol 002** — new "Runtime Behavior Coverage Check" subsection added to Stage 5 gate criteria, placed near the existing minimum-scenario-count rules (one Success + one Failure + one Edge case per BR).
2. **README.md** — Protocol 002 entry updated to reflect the new gate criterion.
3. **Optional**: a template `templates/test-spec-with-e2e.md` showing the canonical E2E scenario shape (entry point / exit point / cross-component path / observable signal).

## Read Before Editing

- `protocols/002-test-driven-business-rules.md` — Stage 5 gate criteria; the natural insertion point for the new rule.
- `protocols/001-documentation-driven-development.md` — Stage 5 gate parent context.
- `prompts/004-create-business-rules.md` — Stage 4 prompt that may need a sibling note ("when authoring a BR that specifies runtime behavior, expect Stage 5 to require an E2E scenario — design your normative statement so it is testable end-to-end").
- `prompts/005-create-br-test-cases.md` — Stage 5 prompt that should reference the new gate criterion.
- Prompt 019, Prompt 020 — sibling methodology prompts; preserve the family voice.

## Triggering Incident Summary (for evidence)

- **Date**: 2026-05-17, during post-Phase-6 manual test session.
- **Sequence**: BR-MILA-170 specified claude-code provider acceptance in dashboard but explicitly scoped out runtime → BR-MILA-173 (claudeAuth volume mount) covered directory mount only → no integration test exercised `provider: claude-code` end-to-end → Phase 6 closed → manual test hit "could not extract result from claude JSON output" because `~/.claude.json` config file was not mounted → diagnosed as FD-012 → ADR #037 captured the systemic pattern.
- **Cost so far**: stab3 sweep (10 dispatches) was necessary in part because the cross-namespace addressing pattern had the same root cause (cross-component invariants not tested at Stage 5). The fix without this prompt would have required successive stab sweeps for each new detection gap. The prompt would have made the gap visible at the moment BR-170 was reviewed at Stage 5.
