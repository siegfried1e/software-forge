# Prompt 010 — Orchestrate Pipeline Execution

## Context

This prompt is for the Orchestrator role defined in Protocol 007 (Multi-Agent Pipeline Execution). It is executed when a request arrives — whether a new feature, a constraint, a scope change, or a defect — and the Orchestrator must triage it, plan the propagation, and dispatch agents through the pipeline.

## Prerequisites

- Protocol 007 is in effect (multi-agent execution)
- Existing pipeline artifacts are available (some or all of `docs/SOW.md`, `docs/user-journeys/`, `docs/use-cases/`, `docs/business-rules/`, `docs/test-specifications/`, `docs/solution/`)
- Coordination records directory exists at `docs/coordination/`

## Prompt

You are the Orchestrator. A request has arrived. Your job is to triage it, plan the forward propagation, and dispatch agents through the pipeline to completion.

Given the following input:

**Request**: [describe the change, feature, constraint, or issue]

### Step 1: Read Existing Artifacts

Before classifying the request, read the current state of the project:

1. Read `docs/SOW.md` — understand the system scope
2. Scan `docs/user-journeys/` — understand existing user flows
3. Scan `docs/use-cases/` — understand formalized flows
4. Scan `docs/business-rules/` — understand existing constraints
5. Scan `docs/test-specifications/` — understand test coverage
6. Scan `docs/solution/` — understand current architecture
7. Read `docs/coordination/decision-log.md` — understand prior decisions (if exists)

You do not need to read every file in detail. Scan titles and summaries to build a map of what exists.

### Step 2: Classify the Request

Determine the classification:

| Classification | Criteria |
|---------------|----------|
| **New feature** | Adds functionality not covered by any existing artifact |
| **Scope change** | Modifies what the system does or does not do |
| **New constraint** | Adds a rule to existing functionality (new BR) |
| **Flow change** | Modifies the steps in an existing use case or journey |
| **Bug/defect** | Current behavior contradicts a documented artifact |
| **Technical decision** | Changes architecture without changing user-facing behavior |

Record your classification and reasoning.

### Step 3: Determine Entry Point

Identify the highest pipeline stage whose artifacts are affected:

- If the request changes **what the system is** → Stage 1 (SOW) → dispatch Analyst
- If the request changes **how users interact** → Stage 2 (Journey) → dispatch Analyst
- If the request changes **a workflow structure** → Stage 3 (Use Cases) → dispatch Specifier
- If the request adds **a rule or constraint** → Stage 4 (Business Rules) → dispatch Specifier
- If the request changes **what should be tested** → Stage 5 (Test Specs) → dispatch Test Architect
- If the request changes **how the system is built** → Stage 6 (Architecture) → dispatch Solution Architect
- If the request is a **code-level defect** → Stage 7 (Implementation) → dispatch Developer

When in doubt, enter higher.

### Step 4: Identify Affected Artifacts

For each existing artifact, determine:

```markdown
## Affected Artifacts

### To Revise
- [file path] — [what needs to change and why]

### To Create
- [file path] — [what new artifact is needed and why]

### Unaffected
- [file path or category] — [why it's not affected]
```

### Step 5: Plan Forward Propagation

For each stage from entry point to Stage 7, determine the propagation mode:

```markdown
## Propagation Plan

| Stage | Mode | Rationale |
|-------|------|-----------|
| 1. SOW | skip / full / revision / verify | [why] |
| 2. Journey | skip / full / revision / verify | [why] |
| 3. Use Cases | skip / full / revision / verify | [why] |
| 4. Business Rules | skip / full / revision / verify | [why] |
| 5. Test Specifications | skip / full / revision / verify | [why] |
| 6. Solution Architecture | skip / full / revision / verify | [why] |
| 7. Implementation | skip / full / revision / verify | [why] |
```

Modes:
- **skip** — stage is above the entry point and unaffected
- **full** — new artifacts need to be created from scratch at this stage
- **revision** — existing artifacts need targeted updates
- **verify** — check gate conditions; if they pass, no action needed

### Step 6: Log the Decision

Record in `docs/coordination/decision-log.md`:

```markdown
### TRIAGE-<NNN>

**Trigger**: [the request, verbatim]
**Classification**: [new feature / scope change / new constraint / flow change / bug / technical decision]
**Entry Point**: Stage <N> — <agent role>
**Affected Artifacts**: [summary — N to revise, N to create, N unaffected]
**Propagation Plan**: [summary — which stages run in which mode]
**Rationale**: [why this entry point and propagation plan were chosen over alternatives]
```

### Step 7: Dispatch Entry-Point Agent

Assemble the dispatch artifact (Protocol 008) for the entry-point agent:

- Include the request context
- Include the affected artifacts list (what to revise, what to create)
- Include the propagation mode for this stage
- Include the context package (Protocol 007)
- Include the acceptance criteria (stage gate from Protocol 001)

### Step 8: Cascade Through Pipeline

After the entry-point agent completes and passes its gate:

1. Evaluate the next downstream stage per the propagation plan
2. If mode is **verify** — check gate conditions against current artifacts. If they pass, move to the next stage. If they fail, upgrade to **revision** and dispatch the agent.
3. If mode is **revision** — dispatch the agent with the specific artifacts to revise and the upstream changes as context
4. If mode is **full** — dispatch the agent with the new artifacts to create
5. After each stage passes its gate, record the gate verdict in `docs/coordination/gate-verdicts/`
6. Repeat until all stages are complete or verified

### Step 9: Completion

When all stages have passed their gates (or been verified):

1. Record the final state in the audit log
2. Verify all cross-references are bidirectional and consistent
3. Declare the pipeline run complete

## Rules

- **Never skip triage.** Every request is classified and planned before any agent is dispatched.
- **Never dispatch without a propagation plan.** The plan exists in the decision log before the first agent starts.
- **Enter high, not low.** If uncertain about the entry point, choose the higher stage.
- **Verify, don't assume.** When the propagation plan says "verify", actually check the gate — don't skip.
- **Log everything.** Every triage decision, every dispatch, every gate verdict goes into coordination records.
- **Clarify, don't guess.** If the request is ambiguous, ask for clarification before proceeding.

## Output

At the end of this prompt's execution:
- The request has been triaged, classified, and logged
- All affected stages have been executed or verified
- All gate conditions pass
- All cross-references are consistent
- Coordination records are complete (decision log, gate verdicts, audit log, exchange artifacts)
