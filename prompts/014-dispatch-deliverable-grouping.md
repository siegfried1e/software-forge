# Prompt 014 — Add Deliverable Grouping Guidance to Protocol 008

## Context

Protocol 008 defines the Dispatch Artifact format, including a deliverables field that lists what the agent must produce. In practice, when the artifact count is high (>15 files), agents exhibit **attention degradation**: early artifacts are produced carefully, late artifacts are compressed or thinner.

**Evidence from a high-count Test Architect dispatch:**

A dispatch instructed the Test Architect to produce 37 test specification files from a flat list. The Orchestrator observed that in prior high-count dispatches (37 BRs, 26 test specs), quality tends to be highest for the first group of artifacts and lowest for the last. The root cause is cognitive: the agent context-switches between unrelated domains (e.g., "retry logic" → "CLI exit codes" → "file-read path traversal") within a single attention pass, and the later artifacts compete for diminishing context space.

**Root cause**: the flat-list deliverable format in Protocol 008 gives the agent no signal about when to "finish one subject and start fresh on the next." The agent treats it as a marathon rather than a series of focused sprints.

## Prompt

Add a **Deliverable Grouping** subsection to Protocol 008's Dispatch Artifact section. The addition should:

1. **Define the problem**: when artifact count exceeds ~15, a flat deliverable list leads to attention degradation — early artifacts are more thorough than late ones.

2. **Define the solution**: the Orchestrator SHOULD group deliverables by subject (component, subsystem, or logical domain) and instruct the agent to complete each group fully before starting the next.

3. **Provide a concrete example**: show a flat-list dispatch vs. a grouped dispatch for the same set of artifacts, and explain why the grouped version produces more consistent quality.

4. **Define when NOT to group**: small counts (<15), single-component dispatches, or homogeneous artifacts where all items share the same context. Grouping adds overhead without benefit in these cases.

5. **Define the Orchestrator's responsibility**: when producing a dispatch with >15 deliverables, the Orchestrator partitions them into groups, orders the groups logically (foundational first, dependent later), and includes the instruction "Complete all artifacts for each group before moving to the next."

6. **Note on partial delivery**: grouped deliverables make partial delivery reportable — an agent can say "Groups 1-4 complete, Group 5 blocked" and the Orchestrator can re-dispatch just the remaining groups. This is harder with a flat list where gaps are scattered.

## Expected Output

An update to Protocol 008 (new subsection under "Dispatch Artifact" or as a new rule). The addition should be concise (under 40 lines) — it's a refinement, not a new protocol.

## Read Before Editing

- `protocols/008-agent-communication-contract.md` — Dispatch Artifact section, deliverables field
- `protocols/007-multi-agent-pipeline-execution.md` — Rule 4 (Context packages are curated) — the grouping principle is an extension of "curated, not dumped"

## Scope Notes

- Do NOT change the required fields of the Dispatch Artifact — deliverable grouping is advisory (SHOULD), not mandatory (MUST). Small dispatches should not be forced to group.
- The grouping is a presentation concern, not a schema change. The deliverables field can still be a flat list when grouping is unnecessary.
- This applies to any agent receiving a high-count dispatch (Specifier, Test Architect, Developer), not just one role.
