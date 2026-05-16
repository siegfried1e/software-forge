# Prompt 017 — Add Context Reset Guidance to Protocol 008

## Context

When the Orchestrator dispatches an agent that has been used in a prior phase or dispatch, the agent's session may retain stale context from the previous work. This stale context competes for attention with the new dispatch, leading to:

- The agent following old task lists instead of the new dispatch
- Confusion between old and new file paths, BR numbers, or design decisions
- The agent applying stale workarounds that are no longer needed

**Evidence from successive phase transitions:**

- An integrator dispatch (a targeted re-validation) kept executing the prior dispatch's broader build plan despite being redirected. Required an explicit `/clear` from the user to break the loop.
- An Analyst transitioning from one phase's SOW to the next would have carried prior-phase SOW context into a completely different scope without a reset.

**Root cause**: Protocol 008's Dispatch Artifact defines what the agent should do, but not how the agent's session should be prepared before reading the dispatch. Context reset is a transport-level concern that the protocol doesn't address.

## Prompt

Add a **Context Reset** field or guidance to Protocol 008's Dispatch Artifact section. The addition should:

1. **Define a `Context Reset` field** (optional, in the dispatch artifact): `reset` or `continue`. Default: `reset` for new phase dispatches, `continue` for targeted revisions within the same scope.

2. **When `reset`**: the Orchestrator's dispatch instructions to the human (or future automated dispatcher) should include a context clear command before the dispatch execution command. In Claude Code, this is `/clear`. In other transports, the equivalent is starting a fresh session.

3. **When `continue`**: the agent reuses its existing session context. Used when the dispatch is a follow-up to a prior dispatch in the same scope (e.g., a gate feedback dispatch followed by its revision within the same scope).

4. **Guidance for the Orchestrator**: when writing dispatch instructions (the text the human copies into the agent's terminal), include the reset command as part of the prompt when `reset` applies:
   ```
   /clear
   Execute D-NNN: docs/coordination/exchanges/D-NNN-dispatch-<role>-<scope>.md
   ```

5. **The default should be `reset`** unless the Orchestrator has a specific reason to continue (e.g., the agent needs prior conversation context to understand a revision). When in doubt, reset — a fresh start with a complete dispatch artifact is more reliable than hoping the agent's prior context is compatible.

## Expected Output

An update to Protocol 008 — either a new optional field in the Dispatch Artifact schema, or a new subsection under "Dispatch Artifact" covering context preparation. Concise (under 20 lines). The key message: dispatches should specify whether the receiving agent needs a clean context, and the default is yes.

## Read Before Editing

- `protocols/008-agent-communication-contract.md` — Dispatch Artifact required and optional fields
- `protocols/007-multi-agent-pipeline-execution.md` — Orchestrator's dispatch responsibilities

## Scope Notes

- This is transport-aware but transport-agnostic. `/clear` is the Claude Code command; other agent runtimes may have different reset mechanisms. The protocol should describe the intent (fresh context), not the specific command.
- Do NOT make this a required field — it's guidance. Many dispatches (especially targeted revisions) work fine with continued context.
