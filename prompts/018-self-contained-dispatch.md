# Prompt 018 — Self-Contained Dispatch Requirement

## Context

Prompt 017 added a `Context Reset` field to Protocol 008's Dispatch Artifact. When set to `reset`, the agent clears its prior session context before reading the dispatch. This solves the stale-context problem — but creates a new one: **the dispatch must now carry ALL information the agent needs**, because the agent remembers nothing.

**Evidence from a phase transitioning to a new technology stack:**

A Solution Architect dispatch was the first written with `Context Reset: reset` after a stack change. The dispatch mentioned the new language and the new messaging library in scattered locations but did not have a consolidated technology stack section. If the Solution Architect had executed with a cleared context, it might have:
- Used the prior phases' default language instead of the new one
- Chosen the wrong HTTP framework for the new language
- Missed serialization conventions required by the target runtime
- Used the wrong CLI framework

The Orchestrator caught this before dispatch and added an explicit technology stack table. But this was a manual save — the protocol should require it.

## Prompt

Add a **Self-Contained Dispatch** rule to Protocol 008, linked to the Context Reset field. The rule should:

1. **Define the principle**: when `Context Reset: reset` is specified (or when dispatching to an agent for the first time), the dispatch artifact MUST be self-contained. The agent should be able to execute the dispatch with NO prior knowledge beyond what's in the dispatch itself and the files it references.

2. **Define what "self-contained" means in practice**: the dispatch MUST include (either inline or via explicit file references) all of the following that are relevant:
   - **Technology stack**: languages, frameworks, libraries, crate/package names, versions if material
   - **Conventions**: naming conventions, file structure patterns, serialization attributes, error handling patterns
   - **Prior decisions**: any architectural or design decisions from earlier phases that constrain this dispatch (with rationale, not just the decision)
   - **Project state**: what exists, what's deployed, what's been built — enough for the agent to understand what it's extending or modifying
   - **Terminology**: project-specific vocabulary that a fresh agent wouldn't know (e.g., domain-specific role names, "formalization debt", "Phase N stabilization")

3. **Define when self-containment is NOT required**: when `Context Reset: continue` is specified, the dispatch can reference prior conversation context and omit information the agent already knows. This is the lightweight path for follow-up dispatches within the same scope.

4. **Practical guidance for the Orchestrator**: when writing a dispatch with `reset`, review the dispatch from the perspective of an agent who has never seen this project. If any sentence requires context not present in the dispatch or its referenced files, either add the context inline or add a file reference. A useful test: "Could a new team member execute this dispatch on their first day, given only the dispatch and its references?"

## Expected Output

An update to Protocol 008 — either amend the Context Reset subsection (from Prompt 017) to include self-containment requirements, or add a new subsection "Self-Contained Dispatches" nearby. Concise (under 25 lines). The key message: a reset dispatch is a complete briefing, not a delta from prior context.

## Read Before Editing

- `protocols/008-agent-communication-contract.md` — Context Reset field (from Prompt 017), Dispatch Artifact schema
- `protocols/007-multi-agent-pipeline-execution.md` — Orchestrator Responsibility 3 (Context package assembly)

## Scope Notes

- This does NOT mean every dispatch must repeat the entire project history. It means every dispatch must reference the right files and include enough inline context that the agent can orient itself. File references are the primary mechanism; inline context is the supplement.
- Do NOT make this retroactive — existing dispatches written before this rule are not non-compliant.
