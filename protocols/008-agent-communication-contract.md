# Protocol 008 — Agent Communication Contract

## Status

Active

## Purpose

Define the concrete communication contract between the Orchestrator and agents during pipeline execution. Protocol 007 defines agent roles, context packages, and gate conditions, but not the artifact format for dispatch, completion, and gate feedback. This protocol fills that gap — it specifies the three communication artifacts that flow between the Orchestrator and agents, independent of how they are delivered.

## The Problem

Protocol 007 establishes that all inter-agent communication passes through the Orchestrator (Rule 5) and that context packages are curated (Rule 4). But it does not define:

- What exactly the Orchestrator sends when dispatching an agent
- What exactly the agent sends back when done
- What exactly the Orchestrator sends when evaluating a gate

Without a standard contract, each pipeline run invents its own handoff format. This leads to:

- **Inconsistent dispatches** — agents receive different levels of detail depending on who assembled the context package
- **Ambiguous completion signals** — the Orchestrator cannot tell if the agent finished, got stuck, or produced partial output
- **Lost gate feedback** — agents receive vague "try again" instructions instead of actionable deficiency lists
- **Unauditable exchanges** — coordination records cannot be standardized because the underlying artifacts have no fixed structure

## Position in the Pipeline

```
                    Dispatch            Completion          Gate Feedback
Orchestrator ──────────────────► Agent ──────────────► Orchestrator ──────────────► Agent
                 (mandate)            (delivery)           (verdict)
                    │                     │                     │
                    └─────────────────────┴─────────────────────┘
                              Protocol 008 governs
```

This protocol operates within Protocol 007. It does not change agent roles, stage ownership, or gate conditions — it standardizes the format of exchanges between agents and the Orchestrator.

## Communication Artifacts

Three artifacts define the contract. Each has required fields that any implementation must satisfy, regardless of transport mechanism.

### 1. Dispatch Artifact (Orchestrator → Agent)

The mandate. This is what the agent reads to know what to do.

**Required fields:**

| Field | Description |
|-------|-------------|
| **Dispatch ID** | Unique identifier for this dispatch (e.g., `D-001`) |
| **Agent Role** | Which role is being dispatched (Analyst, Specifier, Test Architect, Solution Architect, Developer) |
| **Assigned Stages** | Which pipeline stages the agent must execute |
| **Pipeline State** | Current state — which stages are complete, which gates have passed |
| **Context Package** | The artifacts the agent needs, following Protocol 007's context package format |
| **Acceptance Criteria** | The gate conditions the agent's output must satisfy, copied verbatim from the relevant protocol |
| **Constraints** | Any additional constraints — technology stack, scope limitations, prior decisions that must be respected |
| **Working Directory** | The absolute or relative path to the agent's working directory. For parallel Developer dispatches, this is the worktree path created by the Orchestrator (Protocol 007, Workspace Isolation). For serial dispatches, this is the main repository root. The agent MUST operate exclusively within this directory. |
| **Context Reset** | *(Optional)* One of: `reset` or `continue`. Default: `reset`. Instructs the human (or automated dispatcher) whether to start a fresh agent session before executing this dispatch. See Context Reset below. |
| **Prior Gate Feedback** | If this is a re-dispatch after a failed gate, the previous gate feedback artifact (so the agent knows what to fix) |

### Context Reset

An agent dispatched into a session that retains context from prior work may follow old task lists, confuse file paths or BR numbers from a previous scope, or apply stale workarounds that are no longer relevant. The Context Reset field tells the dispatcher whether to start a fresh session.

| Value | Meaning | When to use |
|-------|---------|-------------|
| `reset` *(default)* | Start a fresh session before executing the dispatch | New phase, new scope, or any dispatch where prior session state could interfere |
| `continue` | Reuse the existing session | Follow-up to the immediately prior dispatch in the same scope — e.g., a gate revision (dispatch → its revision) or a continuation within the same integration effort |

**When in doubt, use `reset`.** A complete dispatch artifact provides everything the agent needs; prior context is a liability, not an asset, when the scope changes.

**Dispatcher guidance**: when `reset` applies, include the context clear command before the dispatch instruction. Example for Claude Code:

```
/clear
Execute D-NNN: docs/coordination/exchanges/D-NNN-dispatch-<role>-<scope>.md
```

Other agent runtimes may use different reset mechanisms (fresh API session, new subprocess). The intent is the same: the agent reads only the dispatch artifact and the artifacts it references, not residual conversation state.

### Self-Contained Dispatches

A dispatch with `Context Reset: reset` clears prior session state entirely — the agent remembers nothing. This means the dispatch itself must supply everything the agent needs to execute correctly.

**When `reset` is set (or when dispatching an agent for the first time), the dispatch MUST be self-contained.** The agent must be able to execute with no prior knowledge beyond the dispatch and the files it explicitly references.

A self-contained dispatch includes (inline or via file reference) all of the following that are relevant to the assigned work:

| Category | Examples |
|----------|---------|
| **Technology stack** | Languages, frameworks, libraries, crate/package names, versions if material |
| **Conventions** | Naming conventions, file structure, serialization attributes, error handling patterns |
| **Prior decisions** | Architectural or design decisions from earlier phases that constrain this dispatch — with rationale, not just the decision |
| **Project state** | What exists, what's deployed, what's been built — enough to understand what the agent is extending or modifying |
| **Terminology** | Project-specific vocabulary a fresh agent wouldn't know |

**When self-containment is NOT required**: `Context Reset: continue` dispatches. These are follow-ups within the same scope where the agent's existing session context is intentional. They may reference prior conversation context and omit information the agent already has.

**Orchestrator review test**: before sending a `reset` dispatch, read it from the perspective of an agent who has never seen the project. If any sentence requires context not present in the dispatch or its referenced files, add the context inline or add a file reference. Ask: *could a new team member execute this dispatch on their first day, given only the dispatch and its references?*

File references are the primary mechanism — point to existing artifacts rather than duplicating their content. Inline context is the supplement for decisions and conventions not captured elsewhere.

### Deliverable Grouping

When a dispatch contains more than ~15 deliverables, a flat list leads to **attention degradation**: the agent treats the dispatch as a marathon, context-switches between unrelated subjects, and later artifacts tend to be thinner than earlier ones. The root cause is that the flat list gives the agent no signal about when to finish one subject and start fresh on the next.

**When to group**: dispatches with more than ~15 deliverables, or any dispatch where deliverables span multiple components, subsystems, or logical domains.

**When NOT to group**: fewer than ~15 deliverables, single-component dispatches, or homogeneous artifacts that all share the same context (e.g., 12 test specs for the same subsystem).

**The Orchestrator SHOULD**, for high-count dispatches:

1. Partition deliverables by subject (component, subsystem, or logical domain) — group what belongs together.
2. Order groups logically — foundational artifacts first, dependent artifacts later.
3. Include the instruction: "Complete all artifacts for each group before moving to the next."

**Comparison — same 18 test specs, two dispatch formats:**

*Flat list (degrades):*
```
Deliverables:
- BR-APP-001_test-spec.md  (auth)
- BR-APP-010_test-spec.md  (billing)
- BR-APP-002_test-spec.md  (auth)
- BR-APP-011_test-spec.md  (billing)
- BR-APP-020_test-spec.md  (export)
- ... (13 more, interleaved)
```

*Grouped (consistent quality):*
```
Deliverables:
Group 1 — Authentication (BR-APP-001, 002, 003):
  - BR-APP-001_test-spec.md
  - BR-APP-002_test-spec.md
  - BR-APP-003_test-spec.md
Group 2 — Billing (BR-APP-010, 011, 012):
  - BR-APP-010_test-spec.md
  - BR-APP-011_test-spec.md
  - BR-APP-012_test-spec.md
Group 3 — Export (BR-APP-020, 021, 022):
  - ...

Instruction: Complete all artifacts for each group before moving to the next.
```

**Partial delivery**: grouped deliverables make partial completion reportable. An agent can set status `partial` and write "Groups 1–2 complete, Group 3 blocked" in Unresolved Items. The Orchestrator re-dispatches only the remaining groups. With a flat list, gaps are scattered and harder to isolate for re-dispatch.

### 2. Completion Artifact (Agent → Orchestrator)

The delivery signal. This is how the agent reports what it produced.

**Required fields:**

| Field | Description |
|-------|-------------|
| **Dispatch ID** | References the dispatch this completion responds to |
| **Status** | One of: `complete`, `partial`, `blocked` |
| **Produced Artifacts** | List of file paths created or modified, with a one-line description of each |
| **Acceptance Criteria Self-Check** | The agent's own assessment of each acceptance criterion (met/not met/uncertain), with evidence |
| **Unresolved Items** | Anything the agent could not resolve — ambiguities in upstream artifacts, missing information, conflicting requirements |
| **Discovered Requirements** | New requirements or issues discovered during the work that were not in the dispatch — these flow back to the Orchestrator for triage |

**Status definitions:**

- **complete** — the agent believes all acceptance criteria are met and all assigned stages are done
- **partial** — the agent produced some output but could not finish (must explain why in Unresolved Items)
- **blocked** — the agent cannot proceed without upstream changes or Orchestrator decisions (must explain in Unresolved Items)

### 3. Gate Feedback Artifact (Orchestrator → Agent)

The evaluation result. This is how the Orchestrator communicates whether the work passed or needs rework.

**Required fields:**

| Field | Description |
|-------|-------------|
| **Dispatch ID** | References the dispatch being evaluated |
| **Verdict** | One of: `pass`, `fail` |
| **Criteria Evaluation** | Each acceptance criterion evaluated individually — pass/fail with rationale |
| **Deficiency List** | If failed: specific, actionable items the agent must address. Each deficiency references the criterion it violates and describes exactly what is wrong. |
| **Instructions** | If failed: whether the agent should perform a targeted revision (fix specific items) or a full re-run (start the stage over). If passed: what happens next in the pipeline. |

## Transport Agnosticism

This protocol defines the contract — what is exchanged — not the transport — how it is delivered. The same contract applies whether:

- A human copies the dispatch artifact into an agent's session
- An API call sends the dispatch as a structured payload
- A file is written to a shared directory for the agent to read
- A message queue delivers the artifact asynchronously
- Any future mechanism

Implementations MAY define their own transport conventions. The contract requires only that the required fields are present and interpretable by the receiver.

## Default Location Convention

When communication artifacts are persisted as files (recommended for auditability), they SHOULD live in `docs/coordination/exchanges/`:

```
docs/coordination/exchanges/
  D-001-dispatch-analyst.md
  D-001-completion-analyst.md
  D-001-gate-feedback-analyst.md
  D-002-dispatch-specifier.md
  ...
```

The naming convention is `<Dispatch ID>-<artifact type>-<agent role>.md`. Implementations MAY override this location but MUST record the actual location in the coordination audit log (Protocol 007, Rule 10).

## Failure Modes

| Failure | Detection | Response |
|---------|-----------|----------|
| Agent does not respond | Orchestrator sets a deadline in the dispatch; no completion artifact received by deadline | Orchestrator logs a timeout in the audit log, then re-dispatches to the same or a replacement agent |
| Completion artifact is incomplete | Required fields are missing or Status is `complete` but Acceptance Criteria Self-Check has unmet items | Orchestrator rejects the completion, sends gate feedback with `fail` verdict citing the incompleteness |
| Gate feedback is lost | Agent receives a re-dispatch without prior gate feedback reference | Agent MUST request the gate feedback before proceeding; Orchestrator re-sends from coordination records |
| Agent produces output without reading the dispatch | Output does not match assigned stages or ignores constraints | Orchestrator detects mismatch during gate evaluation; sends `fail` verdict with instruction to re-read the dispatch |
| Agent returns `blocked` | Completion artifact has Status `blocked` with Unresolved Items | Orchestrator triages: resolves by updating upstream artifacts (triggering change propagation per Protocol 007), issuing a decision (logged in the decision log), or escalating to a human |

## Integration with Protocol 007

This protocol formalizes artifacts that Protocol 007 already implies:

| Protocol 007 Concept | Protocol 008 Artifact |
|----------------------|----------------------|
| Context package (Context Handoff section) | Becomes the Context Package field in the Dispatch Artifact |
| Gate verdict (Coordination Records table) | Becomes the Gate Feedback Artifact |
| Context package manifest (Coordination Records table) | Is the Dispatch Artifact itself — the manifest is the dispatch |
| Deficiency list (Orchestrator Responsibilities, item 2) | Becomes the Deficiency List field in the Gate Feedback Artifact |
| Completion signal (implicit in Protocol 007) | Formalized as the Completion Artifact |

Coordination records in `docs/coordination/` (Protocol 007) SHOULD reference or include the communication artifacts. The exchange files in `docs/coordination/exchanges/` are themselves coordination records.

## Rules

1. **Three artifacts, always.** Every agent dispatch MUST produce a dispatch artifact, a completion artifact, and a gate feedback artifact. No exchange is informal or undocumented.

2. **Required fields are mandatory.** Implementations MAY add fields but MUST NOT omit required fields from any communication artifact.

3. **Dispatch ID links the exchange.** Every completion artifact and gate feedback artifact MUST reference the dispatch ID they respond to. Orphaned artifacts (no matching dispatch) are invalid.

4. **Status is honest.** An agent MUST NOT report `complete` if any acceptance criterion is uncertain or unmet. `partial` and `blocked` are not failures — they are signals that enable the Orchestrator to act.

5. **Deficiencies are actionable.** A gate feedback artifact with verdict `fail` MUST include a deficiency list where each item is specific enough for the agent to act on without guessing. "Needs improvement" is not a valid deficiency.

6. **Re-dispatch includes prior feedback.** When re-dispatching an agent after a failed gate, the dispatch artifact MUST include the prior gate feedback in the Prior Gate Feedback field. The agent MUST NOT start from scratch without knowing what failed.

7. **Blocked triggers triage, not retry.** When an agent returns `blocked`, the Orchestrator MUST NOT re-dispatch the same work unchanged. The Orchestrator MUST resolve the blocker first (upstream change, decision, or escalation) and then re-dispatch with updated context.

8. **Artifacts are immutable once sent.** A dispatch artifact, once delivered, is not modified. If the Orchestrator needs to change the mandate, it issues a new dispatch with a new Dispatch ID and references the superseded one.

9. **Transport does not alter the contract.** Regardless of delivery mechanism, the required fields and their semantics remain the same. A dispatch sent as a file has the same contract as one sent via an API.

10. **Exchanges are logged.** Every communication artifact MUST be recorded in the coordination records (Protocol 007, Rule 10). If artifacts are persisted as files in `docs/coordination/exchanges/`, they serve as their own log entries.

## Anti-Patterns

| Anti-Pattern | Why It's Wrong | Correct Approach |
|-------------|----------------|------------------|
| Informal dispatch ("just do Stage 3") | No acceptance criteria, no context package, no audit trail | Full dispatch artifact with all required fields |
| Agent reports "done" without Produced Artifacts | Orchestrator cannot evaluate the gate; no traceability | Completion artifact lists every file created or modified |
| Gate feedback says "fail" without deficiency list | Agent cannot fix what it cannot identify | Every `fail` verdict includes specific, actionable deficiencies |
| Re-dispatch without prior gate feedback | Agent repeats the same mistakes; no learning signal | Prior Gate Feedback field is mandatory on re-dispatch |
| Modifying a sent dispatch artifact | Breaks audit trail; agent may have already acted on the original | Issue a new dispatch with a new ID |
| Orchestrator ignores `blocked` status and re-dispatches | Same blocker will cause the same block; wasted work | Triage the blocker first, then re-dispatch with resolution |
| Completion with `complete` status but uncertain criteria | Gate evaluation is unreliable; Orchestrator trusts a false signal | Report `partial` with honest self-check; let the gate evaluation decide |

## Gate

This protocol's gate is satisfied when:

1. Every agent dispatch in the pipeline run has a corresponding dispatch artifact with all required fields.
2. Every dispatch has a matching completion artifact.
3. Every completion artifact has a matching gate feedback artifact.
4. No `fail` verdict was issued without a deficiency list.
5. No re-dispatch was issued without the prior gate feedback in the Prior Gate Feedback field.
6. All communication artifacts are recorded in the coordination records.
