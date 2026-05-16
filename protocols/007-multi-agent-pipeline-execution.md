# Protocol 007 — Multi-Agent Pipeline Execution

## Status

Active

## Purpose

Define how to decompose the Protocol 001 pipeline across multiple specialized agents, each responsible for a subset of stages. A single agent running all seven stages accumulates context debt, loses focus, and cannot exploit the natural parallelism within the pipeline. This protocol assigns agent roles, defines the orchestrator, specifies context handoff formats, and enforces gate conditions at every agent boundary.

## The Problem

Protocol 001 assumes a single executor — one person or one AI agent — runs all seven stages sequentially. This works for small features but breaks down when:

- **Context window saturation.** By Stage 7, a single agent must hold the SOW, user journeys, use cases, business rules, test cases, and solution architecture simultaneously. The accumulated context crowds out working memory and degrades output quality.

- **Skill mismatch.** Stages demand different competencies: strategic analysis (Stages 1-2), formal specification (Stages 3-4), test design (Stage 5), architecture (Stage 6), and disciplined coding (Stage 7). A generalist agent produces acceptable work at each stage; a specialist agent produces excellent work at its stage.

- **Wasted serial time.** Stage 7 implementation of independent business rules can proceed in parallel, but a single agent must serialize them.

- **Change propagation opacity.** When upstream artifacts change, a single agent must remember what to re-check. Multiple agents with explicit handoff contracts make dependencies visible and auditable.

## Position in the Pipeline

```
SOW → Journey → Use Cases → Business Rules → Test Cases → Solution Draft → Implementation
 │         │         │              │              │              │              │
 └─────────┘         └──────────────┘              │              │              │
   Analyst              Specifier             Test Architect  Sol. Architect  Developer(s)
                                                   │              │              │
                    ┌──────────────────────────────────────────────────────────────┐
                    │                      Orchestrator                            │
                    └──────────────────────────────────────────────────────────────┘
```

This protocol wraps around Protocol 001. It does not replace any stage or gate — it defines who executes each stage and how they coordinate.

## Agent Roles

| Agent | Stages | Inputs | Outputs | Skill Profile |
|-------|--------|--------|---------|---------------|
| **Analyst** | 1-2 (SOW, User Journey) | Idea / feature request | `docs/SOW.md`, `docs/user-journeys/*.md` | Strategic analysis, product thinking, narrative writing |
| **Specifier** | 3-4 (Use Cases, Business Rules) | SOW, user journeys | `docs/use-cases/UC-*.md`, `docs/business-rules/BR-*.md` | Formal specification, cross-referencing, rule extraction |
| **Test Architect** | 5 (Test Specifications) | Business rules | `docs/test-specifications/BR-*_test-spec.md` | Test design, boundary analysis, scenario specification (Protocol 002) |
| **Solution Architect** | 6 (Solution Draft) | Test specifications, business rules, technology stack | `docs/solution/*.md`, `docs/contracts/registry.md` (Protocol 006) | System design, interface design, traceability (Protocol 003) |
| **Developer** | 7 (Red-Green-Refactor) | Solution draft, test specifications, use cases, BRs | Test code and working implementation in packages defined by solution draft | Coding, strict TDD cycles (Protocol 004) |
| **Orchestrator** | None (meta) | All artifacts | Gate verdicts, context packages, change notifications | Pipeline management, gate enforcement, dependency tracking |

### Analyst Owns the Master SOW

In multi-component systems (Protocol 006), the Analyst is responsible for maintaining the master SOW. The Orchestrator validates that the master SOW is up to date at the phase gate, but does not author it.

### Solution Architect Owns the Contract Registry

In multi-component systems (Protocol 006), the Solution Architect maintains the Contract Registry (`docs/contracts/registry.md`). When a phase involves cross-project integration (Protocol 005), the Solution Architect creates or updates the contract documents and records them in the registry. The Orchestrator validates that the registry is up to date at the phase gate.

### Stages 3-4 Are Co-Owned

Use cases and business rules have bidirectional cross-references (Protocol 001, Rule 4). Splitting them across two agents would require repeated round-trips to satisfy the cross-reference gate. A single Specifier agent owns both stages, producing use cases and business rules as a cohesive unit. The Specifier MUST NOT hand off Stage 3 output until Stage 4 is also complete and all cross-references are bidirectional.

### Developer Agent Prohibitions

Developer agents (Stage 7) have a narrower write scope than other pipeline agents. Pipeline agents (Analyst, Specifier, Test Architect, Solution Architect) produce pipeline artifacts; their outputs are expected in `docs/`. Developer agents produce code and, optionally, a completion report. They MUST NOT produce coordination records. These prohibitions are behavioral constraints, not merely normative guidelines — they are violations regardless of intent.

#### Prohibition 1: No writes to the coordination directory

Developer agents MUST NOT write to `docs/coordination/` or any of its subdirectories. This includes:

- Gate verdict files (any file named `*gate-verdict*`, `*gate-feedback*`, or equivalent)
- Audit log files (`audit-log.md` or any append to it)
- Change propagation records
- Dispatch artifacts (these are authored by the Orchestrator, not agents)
- Decision log entries

**Rationale**: The coordination directory is the Orchestrator's exclusive domain (Rule 10). When developer agents write verdict files, they produce a false audit signal — the coordination record says PASS while the Orchestrator has not yet evaluated the gate. This creates a safety-critical discrepancy: a broken branch may be merged on the basis of a self-written verdict.

**Why agents do this**: Developer agents receive their dispatch artifact from `docs/coordination/exchanges/`. They observe that every dispatch has a matching completion, gate verdict, and audit log entry — and extrapolate helpfully. The prohibition must be explicit to override this pattern-matching behavior.

**Permitted**: A developer agent MAY write a **completion report** to the path explicitly specified in its dispatch artifact. This is typically a local `reports/` directory inside the worktree (e.g., `reports/D-NNN-completion.md`), never inside `docs/coordination/`.

#### Prohibition 2: No branch merge operations

Developer agents MUST NOT execute:

- `git merge <any-branch>` — merging any branch, including their own into another
- `git checkout <shared-branch>` — switching to any branch other than their dedicated working branch
- `git push` to shared branches (`main`, `develop`, or any branch not exclusively owned by their dispatch)
- Any other operation that affects branches outside their worktree

**Rationale**: The Orchestrator is the sole merge authority. It merges dispatch branches sequentially, runs the full test suite after each merge, and halts on failure. An agent merging its own branch bypasses the gate entirely — if the Orchestrator had failed the gate, the broken code is now on the main branch.

**Permitted**: A developer agent MAY `git add`, `git commit`, and `git push` to its own dedicated branch (`impl/BR-PROJ-NNN`) within its worktree.

#### Prohibition 3: Self-check ≠ gate verdict

Protocol 008 permits an **Acceptance Criteria Self-Check** section in the completion artifact. This is encouraged — the agent reports which criteria it believes it has met, with evidence. However:

| Permitted | Forbidden |
|-----------|-----------|
| "I believe criterion X is met because Y" — inside the completion report | Any file titled or structured as a gate verdict |
| A self-check with `met` / `not met` / `uncertain` per criterion | `Gate PASS` or `Gate FAIL` as a standalone verdict |
| Honest `partial` or `blocked` status when criteria are uncertain | `complete` status with unmet criteria listed as met |

The Orchestrator's gate evaluation is the only valid gate verdict. The self-check is input to that evaluation, not a substitute for it.

#### Enforcement

Projects using Protocol 007 SHOULD reproduce these prohibitions in each developer agent's CLAUDE.md (or equivalent behavior file) so the agent encounters them at session start. A reusable template is provided in `templates/developer-claude-md.md`. Copy it into the project's developer agent configuration and substitute the project-specific fields.

### Orchestrator Prohibitions

The Orchestrator's write scope is exclusively `docs/coordination/` and its subdirectories (Rule 10). This is more than a convention — it is a behavioral constraint enforced by the same logic that governs Developer Agent Prohibitions: a correct undocumented change is still undocumented, and traceability is the pipeline's core value.

#### Prohibition: No writes to project directories

The Orchestrator MUST NOT create, modify, or delete files in directories owned by pipeline agents — source code, documentation, configuration, Helm charts, manifests, or any other file tracked in git outside of `docs/coordination/`.

This applies regardless of how small or obvious the fix is. A one-line change that bypasses a dispatch has no BR, no test specification, no gate, and no audit trail. If the change is ever regenerated from specifications, the fix disappears. If it introduces a regression, there is no test to catch it.

**The "small fix" temptation**: the Orchestrator is in a privileged position — it evaluates every agent's output and can often see exactly what needs changing. This visibility is necessary for gate evaluation; it is not a mandate to apply fixes. When the Orchestrator identifies a defect during gate evaluation or integration monitoring, the correct action is to dispatch the owning agent with a targeted fix dispatch. The dispatch MAY be lightweight (minimal context, single-issue scope), but it MUST exist.

#### Permitted actions

| Action | Permitted |
|--------|-----------|
| Read any file in any directory | Yes — full read access is required for gate evaluation |
| Run verification commands (`cargo test`, `helm lint`, `kubectl get`, `npm test`) | Yes — operational validation, not source modification |
| Write to `docs/coordination/` and subdirectories | Yes — audit log, decision log, gate verdicts, dispatches, planning documents |
| Apply an existing, git-tracked file to a cluster (`kubectl apply -f crd.yaml`) | Yes — operational action; the file itself is not modified |
| Create or modify any file tracked in git outside `docs/coordination/` | No |

#### The source / operational boundary

The distinction is whether a git-tracked file is modified:

- **Operational**: `kubectl apply -f existing-file.yaml`, `kubectl scale`, `kubectl set env`, `helm upgrade` using existing chart files — these deploy or configure using files that already exist in the repository. No file is modified.
- **Source**: editing `values.yaml`, adding a CRD YAML, modifying `TransportStep.tsx`, patching any file in-place — these change what is tracked in git and require a dispatch.

When in doubt: if `git status` would show the change, it is a source change and requires a dispatch.

### Agent Behavior on Undocumented Architecture

When ANY agent — Orchestrator, Analyst, Specifier, Test Architect, Solution Architect, or Developer — encounters an architectural choice that is referenced by upstream artifacts (code, charts, BRs, etc.) but cannot be traced to an authoritative document (ADR, decision-log entry, SOW section, solution doc, or BR), the agent MUST:

1. **Stop.** Do not assume the rationale. Do not invent a justification "that sounds right".
2. **Flag the gap.** In the completion report, list the undocumented decision under a `Discovered Architectural Gaps` section.
3. **Choose an action**:
   - **(a) Ask** — if the agent has a synchronous channel (e.g., the Orchestrator in chat), surface the question before proceeding.
   - **(b) Report blocked** — if the agent operates asynchronously and the gap blocks its deliverables, set status to `blocked` and list the gap.
   - **(c) Proceed with explicit assumption** — only if the dispatch's acceptance criteria permit it and the assumption is clearly marked `ASSUMPTION (pending ADR): …` in the output artifact.

**The invent signal.** The phrase "I think the reason is…" or "this is probably because…" in an agent's reasoning is a signal that the agent is fabricating a rationale rather than citing a documented one. The correct response to that signal is to stop and apply step 1.

**Orchestrator-specific responsibility.** When the Orchestrator discovers an undocumented architectural decision during gate evaluation, completion-report review, or any other activity:

- Write a retroactive ADR in `docs/coordination/` OR ask the user for the rationale before proceeding.
- Do NOT dispatch downstream agents against undocumented architecture. Dispatching agents that must infer the missing rationale propagates the gap downstream.
- Record the retroactive ADR in the decision log with a note that the decision was undocumented at the time it was first applied.

**Why this matters.** When architectural decisions are implicit, each agent infers the rationale independently. Inferred rationales drift: ten agents produce ten slightly different versions, and the one that propagates downstream may not match the original intent. Mismatch accumulates until integration testing surfaces it as a discrepancy (e.g., a BR specifying resource placement that contradicts the chart's design). A single documented decision stays consistent indefinitely. See Protocol 001 (Architectural Decision Records) for the ADR requirement.

### Stage 7 Parallelization

The Orchestrator MAY spawn multiple Developer agents to work on independent business rules concurrently. Two BRs are independent when neither's implementation imports or calls the other's enforcing component (as defined in the traceability matrix). The Orchestrator determines independence by reading `docs/solution/traceability.md` and the dependency order from Protocol 004, Rule 6.

Each parallel Developer:

- Receives the same solution draft and its assigned BR subset
- Works in its own branch
- Follows Protocol 004 strictly (one test at a time)
- Reports completion to the Orchestrator

The Orchestrator merges branches and runs the full test suite before declaring Stage 7 complete.

## The Orchestrator

The Orchestrator is a coordinating agent that does not produce pipeline artifacts. It enforces sequencing, evaluates gates, assembles context packages, and propagates changes.

### Responsibilities

1. **Sequencing.** Ensure agents execute in pipeline order. Do not dispatch the Specifier until the Analyst's gate passes. Do not dispatch the Test Architect until the Specifier's gate passes.

2. **Gate Enforcement.** After each agent completes, the Orchestrator evaluates the stage gate conditions from Protocol 001. If a gate fails, the Orchestrator returns the work to the responsible agent with a specific deficiency list. The Orchestrator MUST NOT advance the pipeline past a failed gate.

3. **Context Package Assembly.** Before dispatching an agent, the Orchestrator assembles a context package (see Context Handoff) containing exactly the artifacts the agent needs — no more, no less.

4. **Parallel Dispatch.** When Stage 7 permits parallelism, the Orchestrator partitions BRs into independent groups, spawns Developer agents, and manages branch merging.

5. **Change Propagation.** When an upstream artifact changes after its gate was passed, the Orchestrator identifies all downstream agents affected and triggers re-evaluation (see Change Propagation).

6. **Completion Declaration.** The Orchestrator declares the pipeline complete only when Stage 7's gate passes and all tests are green.

7. **Request Triage.** When a new request arrives, the Orchestrator classifies it, determines the pipeline entry point, identifies affected artifacts, and plans the forward propagation. See Request Triage below.

8. **Formalization Debt Tracking.** When any change is applied outside the Protocol 001 pipeline (ad-hoc fixes during integration, operational patches), the Orchestrator MUST record it as formalization debt in the decision log and ensure it is resolved before the next phase's Stage 7 begins. See Protocol 006, Rule 9.

## Request Triage

Any request — new feature, technical constraint, scope change, bug fix — enters through the Orchestrator. The Orchestrator MUST NOT forward a request directly to an agent without first classifying it and planning the propagation.

### Step 1: Classify the Request

| Classification | Description | Example |
|---------------|-------------|---------|
| **New feature** | Adds functionality not covered by existing artifacts | "Add an ingress controller" |
| **Scope change** | Modifies the boundaries of what the system does | "Remove multi-tenant support" |
| **New constraint** | Adds a rule or requirement to existing functionality | "All endpoints MUST require TLS" |
| **Flow change** | Modifies how an existing feature works | "Users should authenticate via OAuth instead of API key" |
| **Bug/defect** | Existing behavior does not match documented specification | "The rate limiter allows 200 req/s but BR-PROJ-012 says 100" |
| **Technical decision** | Impacts architecture without changing user-facing behavior | "Switch from PostgreSQL to CockroachDB" |

### Step 2: Determine Entry Point

The entry point is the **highest stage** whose artifacts are affected by the request:

| If the request affects... | Entry point | Agent dispatched |
|--------------------------|-------------|-----------------|
| What the system is or does | Stage 1 (SOW) | Analyst |
| How users interact with the system | Stage 2 (User Journey) | Analyst |
| The structure of a workflow or flow | Stage 3 (Use Cases) | Specifier |
| An enforceable rule or constraint | Stage 4 (Business Rules) | Specifier |
| What should be tested (not how) | Stage 5 (Test Specifications) | Test Architect |
| How the system is built | Stage 6 (Solution Architecture) | Solution Architect |
| Code that doesn't match its specification | Stage 7 (Implementation) | Developer |

When in doubt, enter higher. Entering too low risks artifacts drifting out of alignment.

### Step 3: Identify Affected Artifacts

Before dispatching the entry-point agent, the Orchestrator reads the existing artifacts and determines:

- **Which artifacts need revision** — existing documents that the request changes
- **Which artifacts need creation** — new documents that don't exist yet
- **Which artifacts are unaffected** — existing documents that remain valid

This assessment is included in the dispatch artifact (Protocol 008) so the agent knows its scope.

### Step 4: Plan Forward Propagation

From the entry point, every downstream stage MUST be evaluated. For each downstream stage, the Orchestrator determines:

| Propagation mode | When to use | What happens |
|-----------------|-------------|-------------|
| **Full run** | New artifacts created upstream that didn't exist before | Agent executes the stage from scratch for the new artifacts |
| **Targeted revision** | Existing artifacts modified upstream | Agent revises only the affected sections, preserving unaffected content |
| **Verification only** | Upstream change is minor and may not affect this stage | Orchestrator checks the gate conditions; if they still pass, skip the stage |

The propagation plan is recorded in the decision log (coordination records) before any agent is dispatched.

### Step 5: Execute

The Orchestrator dispatches agents in pipeline order starting from the entry point. Each dispatch includes:

- The request classification and context
- Which artifacts to create or revise (from Step 3)
- The propagation mode for this stage (from Step 4)
- The full context package (Protocol 007, Context Handoff)
- Acceptance criteria (the stage gate)

After each stage completes and passes its gate, the Orchestrator evaluates whether the next stage needs a full run, targeted revision, or verification only — and dispatches accordingly.

### Triage Rules

1. **Every request goes through triage.** No agent receives a request directly from outside the pipeline. The Orchestrator is the single entry point.

2. **Entry point is the highest affected stage.** If a request affects both a use case and a business rule, enter at Stage 3 (Use Cases), not Stage 4.

3. **Forward propagation is mandatory.** Every stage downstream of the entry point MUST be at least verified. Skipping downstream stages creates artifact drift.

4. **Triage is logged.** The classification, entry point, affected artifacts, and propagation plan are recorded in the decision log before execution begins.

5. **Ambiguous requests are clarified, not guessed.** If the Orchestrator cannot classify a request or determine the entry point, it requests clarification rather than making assumptions.

6. **Recurring discrepancies MUST be fixed, not worked around.** See Recurring Discrepancies below.

### Recurring Discrepancies

A **recurring discrepancy** is the same issue — same root cause, same component — appearing in two or more completion artifacts, with the same or similar workaround applied each time.

**Detection.** The Orchestrator MUST review the Unresolved Items and Discovered Requirements sections of every completion artifact (Protocol 008) against prior completions. If the same discrepancy appears twice, it triggers this rule.

**Obligation.** When a recurring discrepancy is detected, the Orchestrator MUST NOT accept another workaround. It MUST:

1. **Reclassify** the discrepancy as a bug/defect, regardless of how it was originally classified
2. **Triage** it through the standard process (Step 1-5 above), entering at the stage determined by the root cause
3. **Mark it as mandatory** — the fix dispatch cannot be deferred or deprioritized
4. **Record the pattern** in the decision log with:
   - The dispatch IDs where the discrepancy appeared
   - The workaround that was applied each time
   - The root cause analysis
   - The fix dispatch ID

**Integration with triage.** A recurring discrepancy enters the pipeline like any other triaged request. The only difference is that it is non-deferrable — the Orchestrator MUST dispatch the fix before any new feature work proceeds.

**Prevention.** After fixing a recurring discrepancy, the Orchestrator SHOULD check whether the root cause could affect other components. If so, it creates additional verification dispatches for the affected components.

## Pipeline Artifacts vs. Coordination Records

The Orchestrator does not produce pipeline artifacts but MUST maintain coordination records. These are distinct categories:

**Pipeline artifacts** are the outputs of Stages 1-7 — SOW, user journeys, use cases, business rules, test cases, solution draft, and code. They are authored by the specialized agents (Analyst, Specifier, Test Architect, Solution Architect, Developer). The Orchestrator MUST NOT produce or modify these.

**Coordination records** are meta-documents that only the Orchestrator can write because only the Orchestrator has cross-pipeline visibility. They capture the *why* behind coordination decisions, not just the *what*. They include:

| Record | Purpose | When Written |
|--------|---------|-------------|
| **Audit log** | Who executed what, when, with what result | After every agent dispatch and gate evaluation |
| **Decision log** | Methodology decisions — trigger, problem, resolution, lesson learned | When the Orchestrator makes a judgment call (e.g., targeted revision vs. full re-run) |
| **Gate verdicts** | Pass/fail with detailed rationale and deficiency list | After each gate evaluation |
| **Context package manifests** | What artifacts were sent to each agent and why | Before each agent dispatch |
| **Change propagation records** | What changed upstream, which agents were halted, what was re-evaluated | When upstream artifacts change after their gate passed |

### Location

Coordination records live in `docs/coordination/` within the project:

```
docs/coordination/
  audit-log.md
  decision-log.md
  gate-verdicts/
    stage-1.md
    stage-2.md
    ...
  context-manifests/
    analyst.md
    specifier.md
    ...
  change-propagation/
    <NNN>-<description>.md
```

### The Decision Log

The decision log is to the Orchestrator what the SOW is to the Analyst — it captures the reasoning behind coordination choices. Each entry follows this structure:

```
### <Decision ID>

**Trigger**: What event prompted the decision
**Problem**: What needed to be resolved
**Resolution**: What the Orchestrator decided
**Rationale**: Why this option was chosen over alternatives
**Lesson**: What to carry forward for future pipeline runs
```

## Context Handoff

### Principle

Each agent receives a **context package** — the minimum set of artifacts required for its stages. Agents do not read the full repository. The Orchestrator curates context packages to keep each agent focused and within its context budget.

### Context Packages by Agent

| Agent | Context Package Contents |
|-------|-------------------------|
| **Analyst** | Feature request / idea description. Existing `docs/SOW.md` (if revising). Existing user journeys (if revising). |
| **Specifier** | `docs/SOW.md`, `docs/user-journeys/*.md`. Existing use cases and BRs (if revising). |
| **Test Architect** | `docs/business-rules/BR-*.md`. Existing test specifications (if revising). Protocol 002 rules. |
| **Solution Architect** | `docs/business-rules/BR-*.md`, `docs/test-specifications/BR-*_test-spec.md`, `docs/use-cases/UC-*.md`. Technology stack constraints. Protocol 003 rules. |
| **Developer** | `docs/solution/*.md`, `docs/test-specifications/BR-*_test-spec.md` (assigned subset), `docs/business-rules/BR-*.md` (assigned subset), `docs/use-cases/UC-*.md` (referenced by assigned BRs). Protocol 004 rules. |

### Handoff Format

Each context package is a structured manifest:

```
## Context Package — <Agent Role>

### Pipeline State
Stage: <N>
Prior gates passed: <list>
Assigned stages: <N, N+1>

### Artifacts
<list of file paths with purpose annotations>

### Constraints
<any agent-specific constraints or focus areas>

### Acceptance Criteria
<the gate conditions this agent must satisfy>
```

### Accumulated Context Rule

When the pipeline is long-running and multiple agents have completed, later agents MAY receive a summary of earlier stages rather than the full artifacts. The Orchestrator SHOULD provide:

- Full text of the immediately upstream artifacts (direct inputs)
- Summaries of artifacts two or more stages upstream
- Full text of any artifact explicitly cross-referenced by the direct inputs

## Parallelization

### Where Parallelism Is Permitted

| Opportunity | Condition | Protocol Reference |
|-------------|-----------|-------------------|
| Multiple Developers in Stage 7 | BRs have no implementation dependency on each other per the traceability matrix | Protocol 004, Rule 6 |
| Multiple pipeline runs within a phase | Protocol 006 permits parallel runs when there are no inter-run dependencies | Protocol 006, Rule 3 |

### Where Parallelism Is Forbidden

| Boundary | Why |
|----------|-----|
| Stages 1 → 2 | User journeys depend on the SOW |
| Stages 2 → 3-4 | Use cases formalize journey steps |
| Stages 3-4 → 5 | Tests are derived from business rules |
| Stages 5 → 6 | Solution architecture maps to tests and BRs |
| Stages 6 → 7 | Implementation follows the solution draft |
| Stages 3 and 4 with each other | Bidirectional cross-referencing requires co-authoring |

### Workspace Isolation

Branch isolation alone is insufficient when parallel agents share the same physical working directory. `git checkout` is global to a clone — when one agent switches branches, the working tree changes for every agent sharing that clone. This causes data loss (uncommitted changes overwritten), context corruption (agent reads files from another agent's branch), and merge conflicts on shared files.

**Rule: each parallel agent MUST have its own working directory.** Branch isolation is a necessary but not sufficient condition for parallel work. Workspace isolation ensures that each agent's file reads, writes, and git operations are fully independent.

#### Isolation Mechanisms

| Mechanism | Description | When to Use |
|-----------|-------------|-------------|
| **`git worktree`** (default) | Creates a lightweight working directory linked to the same object store. Each worktree checks out its own branch independently. | Default for all parallel dispatches. Preferred because it shares the object store (no re-clone cost), supports independent branch checkouts, and is cleaned up with a single command. |
| **Separate clone** | A full clone of the repository. Fully independent — separate object store, separate refs. | When worktrees are not supported by the tooling, or when the parallel work requires independent git history operations (e.g., interactive rebase). |

The Orchestrator MUST use `git worktree` unless a specific constraint requires a separate clone. The choice MUST be recorded in the dispatch artifact (Protocol 008) and the decision log.

#### Orchestrator Responsibilities for Workspace Isolation

1. **Create worktrees before dispatch.** Before spawning parallel Developer agents, the Orchestrator creates one worktree per agent:
   ```
   git worktree add ../worktree-BR-PROJ-NNN impl/BR-PROJ-NNN
   ```
   The worktree path and branch name are included in the dispatch artifact (Protocol 008, Working Directory field).

2. **Include the working directory in the dispatch.** The dispatch artifact MUST contain the absolute or relative path to the agent's worktree. The agent operates exclusively within this directory.

3. **Merge from the main repository.** After agents complete, the Orchestrator merges from the main repository (not from within a worktree). The Orchestrator switches to the main clone, merges each agent's branch sequentially, and runs the full test suite after each merge.

4. **Clean up worktrees after merge.** Once a branch is successfully merged and the test suite passes, the Orchestrator removes the worktree:
   ```
   git worktree remove ../worktree-BR-PROJ-NNN
   ```
   If a merge fails, the worktree is preserved until the conflict is resolved.

#### Shared Files

Some files may be written to by multiple agents concurrently (e.g., `docs/coordination/audit-log.md`, shared configuration). Concurrent writes to the same file from different worktrees cause merge conflicts that are difficult to resolve automatically.

**Rule: only the Orchestrator writes to shared files.** Agents MUST NOT write to files that are outside the scope of their assigned BRs. Specifically:

- **Coordination records** (`docs/coordination/*`) — only the Orchestrator writes these (already required by Rule 10).
- **Shared configuration** (e.g., dependency manifests, CI config) — if multiple agents need to modify a shared file, the Orchestrator consolidates the changes after all agents complete. Each agent records its required changes in its completion artifact (Protocol 008, Produced Artifacts field), and the Orchestrator applies them sequentially.
- **Dispatch-scoped files** — each agent writes only to files within the scope of its assigned BRs, as defined by the solution draft and traceability matrix.

If an agent discovers that it needs to modify a file outside its scope, it MUST report this in the Unresolved Items field of its completion artifact and set status to `partial` or `blocked`. The Orchestrator resolves the conflict.

### Parallelism Protocol for Stage 7

1. Orchestrator reads `docs/solution/traceability.md` and builds a BR dependency graph.
2. BRs with no inbound dependencies from other BRs form the first parallel batch.
3. Each Developer in a batch receives its own context package scoped to its assigned BRs.
4. For each Developer in the batch, the Orchestrator creates a worktree and a dedicated branch named `impl/BR-PROJ-NNN` (or `impl/BR-PROJ-NNN-to-MMM` for a group). The worktree path is included in the dispatch artifact.
5. Each Developer works exclusively in its assigned worktree.
6. When all Developers in a batch complete, the Orchestrator merges branches sequentially from the main clone and runs the full test suite after each merge.
7. If a merge introduces test failures, the Orchestrator assigns the conflict to one Developer for resolution.
8. The Orchestrator removes completed worktrees after successful merge.
9. The next batch of BRs (those whose dependencies are now satisfied) is dispatched.

## Change Propagation

### When Upstream Artifacts Change

If a gate-passed artifact is revised (e.g., the SOW changes after Stage 2 is complete), the Orchestrator MUST:

1. **Halt** all downstream agents currently working.
2. **Diff** the changed artifact against the version that downstream agents received.
3. **Assess impact** — determine which downstream artifacts are affected by the change.
4. **Notify** each affected agent with:
   - The diff of the upstream change
   - The specific sections of their output that may need revision
   - Whether a full re-run or a targeted revision is required
5. **Re-gate** — the changed stage's gate MUST be re-evaluated before downstream work resumes.

### Impact Assessment

| Changed Artifact | Affected Stages | Required Action |
|-----------------|-----------------|-----------------|
| SOW | 2, 3-4, 5, 6, 7 | Re-evaluate all downstream; full cascade if scope changed |
| User Journey | 3-4, 5, 6, 7 | Specifier revises affected UCs and BRs; cascade continues |
| Use Case | 4, 5, 6, 7 | Specifier revises affected BRs; cascade continues |
| Business Rule | 5, 6, 7 | Test Architect revises tests; Solution Architect revises affected components |
| Test Case | 6, 7 | Solution Architect verifies traceability; Developer re-runs affected tests |
| Solution Draft | 7 | Developer adapts code to new interfaces |

### Forward-Only Rule

Changes always propagate forward through the pipeline, never backward. If a downstream agent discovers that an upstream artifact is insufficient, it reports to the Orchestrator, which sends the issue back to the upstream agent. The downstream agent does NOT modify upstream artifacts.

## Rules

1. **One dispatch per role.** Each role (Analyst, Specifier, Test Architect, Solution Architect) is filled by exactly one dispatch at a time. The Developer role MAY have multiple concurrent dispatches (see Parallelization).

   An agent is a **dispatch instance** — a session created by the Orchestrator with a specific context package and acceptance criteria (Protocol 008). It is not a persistent identity, a confined workspace, or a long-lived process. Two dispatches are two agents, even if they execute on the same infrastructure or write to the same directory.

   This rule prevents three specific risks:
   - **Self-review** — an agent MUST NOT evaluate output it produced. Gate evaluation is always the Orchestrator's responsibility (Rule 3).
   - **Context bleed** — an agent receives only its context package (Rule 4), not residual state from a prior role's session. A fresh dispatch ensures a clean context boundary.
   - **Accountability opacity** — the audit log (Rule 10) must trace each stage's output to a distinct dispatch ID (Protocol 008). If two roles share a dispatch, the log cannot distinguish who produced what.

   Multiple roles MAY share the same underlying workspace (e.g., all documentation roles write to `docs/`). Directory isolation is not required — dispatch isolation is. Stage 7 Developers additionally require **branch isolation** for parallel work (Rule 7), but this is a parallelization constraint, not an identity constraint.

2. **Orchestrator is mandatory.** A pipeline with multiple agents MUST have an Orchestrator. No agent dispatches itself or decides its own sequencing.

3. **Gates are non-negotiable.** The Orchestrator MUST enforce every gate defined in Protocol 001 (Stages 1-7), Protocol 002 (Stage 5), Protocol 003 (Stage 6), and Protocol 004 (Stage 7). An agent cannot self-certify its own gate. Developer agents in particular MUST NOT write gate verdict files, audit log entries, or any coordination record — doing so produces a false audit signal regardless of whether the self-verdict is correct (see Developer Agent Prohibitions).

4. **Context packages are curated.** The Orchestrator MUST assemble a context package for each agent containing only the artifacts relevant to that agent's stages. Agents MUST NOT be given the entire repository as context.

5. **Agents do not communicate directly.** All inter-agent communication passes through the Orchestrator. Agent A does not send artifacts to Agent B — it delivers them to the Orchestrator, which gates them and then packages them for Agent B.

6. **Specifier owns Stages 3 and 4 together.** Use cases and business rules MUST be produced by the same agent to satisfy bidirectional cross-referencing (Protocol 001, Rule 4). The Specifier does not hand off Stage 3 independently.

7. **Parallel Developers work in isolated workspaces.** Each Developer operates on a dedicated branch in its own worktree (see Workspace Isolation). Branch isolation alone is insufficient — each parallel agent MUST have its own working directory. The Orchestrator creates worktrees before dispatch, includes the worktree path in the dispatch artifact (Protocol 008), manages merging from the main clone, and cleans up worktrees after merge. No Developer pushes to the main branch directly.

8. **Changes halt downstream.** When an upstream artifact changes, the Orchestrator MUST halt affected downstream agents before they produce work based on stale inputs.

9. **No agent skips a stage.** Even in multi-agent execution, the pipeline sequence defined in Protocol 001 is inviolable. Agents execute their assigned stages in order.

10. **Orchestrator maintains coordination records.** The Orchestrator MUST maintain all coordination records in `docs/coordination/`: the audit log, the decision log, gate verdicts with detailed rationale, context package manifests, and change propagation records. These are the only documents the Orchestrator authors — they are coordination records, not pipeline artifacts (see Pipeline Artifacts vs. Coordination Records).

## Anti-Patterns

| Anti-Pattern | Why It's Wrong | Correct Approach |
|-------------|----------------|------------------|
| Single agent runs all 7 stages | Context saturation degrades later stages; no specialization benefit | Assign specialized agents per role |
| Split Stages 3 and 4 across agents | Bidirectional cross-referencing requires constant round-trips; gate cannot be evaluated in isolation | Single Specifier owns both stages |
| Agents communicate peer-to-peer | No gate enforcement; no audit trail; context packages uncontrolled | All communication through Orchestrator |
| Agent self-certifies its gate | Conflict of interest; agent may overlook deficiencies in its own output | Orchestrator evaluates all gates |
| Orchestrator produces pipeline artifacts | Mixes coordination with creation; Orchestrator context becomes bloated | Orchestrator authors coordination records only (audit log, decision log, gate verdicts, manifests) — never pipeline artifacts (SOW, UCs, BRs, tests, code) |
| All BRs parallelized in Stage 7 | Dependent BRs will conflict at merge; shared code modified concurrently | Parallelize only independent BRs per dependency graph |
| Parallel agents share one working directory | `git checkout` is global — branch switch by one agent corrupts all others' working trees; uncommitted changes lost | Each parallel agent gets its own worktree (`git worktree add`) |
| Parallel agents write to shared files | Concurrent writes cause merge conflicts that are hard to resolve automatically | Only the Orchestrator writes to shared files; agents record needed changes in their completion artifact |
| Developer agent writes a gate verdict file | Creates a false audit signal; Orchestrator may never re-evaluate if the coordination record already shows PASS | Only the Orchestrator authors gate verdicts; developer agents write completion reports only |
| Developer agent merges its own branch | Bypasses gate evaluation; broken code lands on main if the Orchestrator had failed the gate | Orchestrator is the sole merge authority; developer agent commits and pushes to its dedicated branch only |
| Full repository given as context | Wastes context budget on irrelevant artifacts; agent loses focus | Curated context packages per agent |
| Downstream agent fixes upstream artifact | Violates ownership; upstream agent loses awareness of the change | Report to Orchestrator; upstream agent revises |
| Skip gate after upstream change | Downstream work based on stale inputs; artifacts drift out of alignment | Re-gate the changed stage; cascade impact assessment |

## Gate

This protocol's gate applies to the pipeline run as a whole:

1. Every stage gate from Protocol 001 has been evaluated and passed by the Orchestrator (not self-certified by agents).
2. Every context handoff between agents has a corresponding manifest in `docs/coordination/context-manifests/`.
3. If Stage 7 was parallelized, all branches are merged, and the full test suite passes on the merged result.
4. No downstream agent operated on stale upstream artifacts (or any staleness was detected, recorded in `docs/coordination/change-propagation/`, and corrected).
5. Coordination records are complete — audit log, decision log, gate verdicts, context manifests, and change propagation records are all present in `docs/coordination/`.
