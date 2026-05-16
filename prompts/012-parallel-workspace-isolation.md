# Prompt 012 — Add Workspace Isolation for Parallel Agents

## Context

Protocol 007 Rule 7 states: "Parallel Developers work on isolated branches. Each Developer operates on a dedicated branch. The Orchestrator manages merging."

In practice, "isolated branches" is insufficient when agents share the same physical repository. `git checkout` is global — switching one agent's branch changes the working directory for all agents. This causes:

- **Data loss**: uncommitted changes from Agent A are overwritten when Agent B switches branches
- **Context corruption**: Agent A reads files from Agent B's branch
- **Merge conflicts**: both agents modify the same files (e.g., shared config, audit logs)

This was discovered during a multi-component phase when operator and dashboard agents were dispatched in parallel on the same repo.

## Prompt

Add a section to Protocol 007 (under Parallelization or as a new subsection) that addresses workspace isolation for parallel agents. The addition should:

1. **Define the problem**: branch isolation ≠ workspace isolation in a single-repo setup
2. **Define the solution**: each parallel agent MUST have its own working directory. Options:
   - `git worktree` — lightweight, shares object store, each worktree has its own branch
   - Separate clones — heavier but fully isolated
   - The protocol should recommend `git worktree` as the default and allow alternatives
3. **Define the Orchestrator's responsibility**: 
   - Create worktrees before dispatching parallel agents
   - Include the worktree path in the dispatch artifact (Protocol 008)
   - Merge from the main repo after agents complete
   - Clean up worktrees after merge
4. **Update Protocol 008** if needed: the dispatch artifact may need a "Working Directory" field
5. **Define what happens to shared files** (e.g., `docs/coordination/audit-log.md` that multiple agents might write to concurrently). Options: only the Orchestrator writes to shared files, or agents write to dispatch-specific files that the Orchestrator consolidates.

Read:
- `protocols/007-multi-agent-pipeline-execution.md` — Rule 7, Parallelization section
- `protocols/008-agent-communication-contract.md` — Dispatch artifact fields

## Expected Output

An update to Protocol 007 (new subsection under Parallelization) and optionally Protocol 008 (new dispatch field). Place changes directly in the protocol files.
