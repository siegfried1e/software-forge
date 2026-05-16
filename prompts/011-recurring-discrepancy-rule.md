# Prompt 011 — Add Recurring Discrepancy Rule to Triage

## Context

Protocol 007's Request Triage section (and Prompt 010) defines how the Orchestrator classifies and routes incoming requests. Protocol 008 defines failure modes for agent communication. Neither addresses a specific failure pattern observed in practice:

**A discrepancy is reported, worked around, and then the same discrepancy recurs in a later dispatch.** The workaround becomes the norm instead of a fix being applied.

Examples from real pipeline execution:
- A NodePort conflict was worked around in one dispatch with `--set service.nodePort=30081`. The same workaround was needed again in a later dispatch.
- An `imagePullPolicy` issue was discovered in a deployment dispatch. Without a rule, it would recur in every future redeploy with the `latest` tag.

The Orchestrator failed to escalate these from "workaround" to "fix" after the first occurrence. A rule is needed.

## Prompt

Add a rule to Protocol 007's Triage section (or a new section) that addresses recurring discrepancies. The rule should:

1. **Define what a recurring discrepancy is** — the same issue appears in two or more dispatch completions, with the same or similar workaround applied each time.

2. **Define the Orchestrator's obligation** — when a discrepancy recurs, the Orchestrator MUST NOT accept another workaround. It MUST:
   - Classify it as a bug/defect (not a one-time environment issue)
   - Create a fix dispatch targeting the root cause
   - Record the pattern in the decision log

3. **Define a detection mechanism** — the Orchestrator SHOULD review the Discrepancies section of every completion artifact against prior completions. If the same discrepancy (same root cause, same component) appears twice, it triggers the recurring discrepancy rule.

4. **Integrate with existing triage** — recurring discrepancies enter the triage at the entry point determined by the root cause, just like any other request. The difference is that they are mandatory (the Orchestrator cannot defer them).

Read:
- `protocols/007-multi-agent-pipeline-execution.md` — Request Triage section
- `prompts/010-orchestrate-pipeline.md` — Orchestrator workflow

## Expected Output

An addition to Protocol 007 (new rule or new subsection under Request Triage) that prevents recurring discrepancies from being perpetually worked around. Place the update directly in `protocols/007-multi-agent-pipeline-execution.md`.
