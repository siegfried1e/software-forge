# Prompt 015 — Formalize Orchestrator Prohibitions

## Context

Protocol 007 defines the Orchestrator's access rights as `Write: docs/coordination/ only`. However, unlike the Developer Agent Prohibitions (added in Prompt 013), this constraint is implied by the access table rather than stated as an explicit behavioral prohibition. In practice, the Orchestrator violated it twice during integration testing:

1. **Direct edit to a UI component** (`TransportStep.tsx`) — added a transport option to a dropdown. One line, correct change, but bypassed the frontend agent entirely. No BR, no test spec, no dispatch.
2. **Direct copy of a CRD YAML** into the operator Helm chart — correct fix for a missing file, but bypassed the operator agent. No dispatch.

Both violations occurred because the fixes were small (1-2 lines) and the Orchestrator judged that a full dispatch was disproportionate overhead. The changes were correct and uncontroversial. But the pipeline's value is not in preventing incorrect changes — it's in maintaining traceability. A correct undocumented change is still undocumented.

**Root cause**: the same pattern that led developer agents to write gate verdicts — "it's small, it's obviously right, and I can see what needs to happen." Prompt 013 added explicit prohibitions for developers. The Orchestrator needs the same treatment.

## Prompt

Add an **Orchestrator Prohibitions** subsection to Protocol 007, parallel to the existing Developer Agent Prohibitions. The addition should:

1. **State the prohibition explicitly**: the Orchestrator MUST NOT create, modify, or delete files in project directories (any directory owned by a Developer, Analyst, Specifier, Test Architect, or Solution Architect). The Orchestrator's write scope is exclusively `docs/coordination/` and its subdirectories.

2. **Cover the "small fix" temptation**: when the Orchestrator identifies a one-line fix during gate evaluation or integration monitoring, the correct action is to dispatch the owning agent — not to apply the fix directly. The dispatch can be lightweight (a targeted fix dispatch with minimal context), but it must exist. The alternative — the Orchestrator editing project files — breaks traceability for the same reason that a developer writing gate verdicts breaks audit integrity.

3. **Permitted actions**: the Orchestrator MAY:
   - Read any file in any directory (full read access is essential for gate evaluation)
   - Run verification commands (`cargo test`, `helm lint`, `kubectl get`) to validate gate criteria
   - Write to `docs/coordination/` (audit log, decision log, gate verdicts, dispatches, planning documents)
   - Apply cluster-level changes that are not source-controlled (e.g., `kubectl apply` a CRD from an existing file, `kubectl scale`, `kubectl set env`) — these are operational, not source changes

4. **The line between operational and source**: modifying a file tracked in git requires a dispatch. Applying a file that already exists in git to a cluster (e.g., `kubectl apply -f crd.yaml`) is an operational action and does not require a dispatch. Creating or modifying the file itself does.

## Expected Output

An update to Protocol 007 — new subsection "Orchestrator Prohibitions" placed after or near the "Developer Agent Prohibitions" subsection. Concise (under 30 lines). Cross-reference the existing access rights table.

## Read Before Editing

- `protocols/007-multi-agent-pipeline-execution.md` — Orchestrator role definition, access rights table, Developer Agent Prohibitions subsection
