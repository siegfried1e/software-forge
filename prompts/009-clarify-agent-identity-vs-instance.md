# Prompt 009 — Clarify Agent Identity vs Instance in Protocol 007

## Context

Protocol 007 Rule 1 states: "One agent per role." Protocol 007 defines 5 producer roles (Analyst, Specifier, Test Architect, Solution Architect, Developer) plus the Orchestrator.

In practice, projects that adopt this protocol face a question Protocol 007 does not answer:

**Does "one agent per role" mean a separate identity (distinct workspace, persistent state, confined scope) or a separate instance (distinct session, fresh context, no shared state with other roles)?**

This matters because:

- Some implementations confine agents to directories. If each role needs its own directory, that's 5+ directories for documentation roles that all write to `docs/`.
- Some implementations use session-based agents. An agent is a fresh session with a mandate — it has no persistent identity beyond the dispatch. Two different roles are just two different sessions.
- Stages 1-6 all produce documentation. The skill profiles overlap significantly (Analyst and Specifier both write markdown; Test Architect and Solution Architect both read BRs). Forcing 4 separate identities for documentation work may create overhead without benefit.
- Stage 7 (Developer) is clearly different — it writes code, may need a build environment, and parallelization demands isolated branches. Separation here is unambiguous.

The underlying concern behind Rule 1 is valid: an agent should not self-review (the Analyst should not evaluate the Analyst's own gate), and context from one role should not bleed into another. But there are multiple ways to achieve this.

## Prompt

Clarify Protocol 007 Rule 1 by addressing these questions:

1. **What does "one agent" mean?** Is it:
   - A persistent identity with its own workspace and state (directory-confined agent)?
   - A fresh instance per dispatch with no state beyond the context package (session-based agent)?
   - Something else?

2. **What problem does Rule 1 solve?** Identify the specific risks that Rule 1 prevents. For example:
   - Self-review (agent evaluates its own output)
   - Context bleed (agent carries assumptions from a previous stage into the current one)
   - Skill dilution (generalist agent does acceptable work instead of specialist doing excellent work)
   - Accountability opacity (can't tell who produced what)

3. **Can multiple roles share a workspace if they are dispatched as separate instances?** If the Analyst and the Specifier are two separate dispatches (different context packages, different acceptance criteria, different sessions) but write to the same `docs/` directory, does that violate Rule 1?

4. **Is the rule different for documentation roles (Stages 1-6) vs code roles (Stage 7)?** Documentation roles all produce markdown in `docs/`. Code roles produce code in project directories. Does the rule apply uniformly, or should it distinguish between these categories?

5. **Update Rule 1** with the clarification. If the rule needs splitting (e.g., one rule for identity separation, another for instance separation), do so. If the rule is fine as-is but needs an explanation section, add that instead.

Read `protocols/007-multi-agent-pipeline-execution.md` (especially the Agent Roles table, Rule 1, and the Anti-Patterns table) before writing.

## Expected Output

An updated Rule 1 (and any supporting sections) for Protocol 007, provided as a diff or replacement text. Not a full protocol rewrite — just the targeted clarification.
