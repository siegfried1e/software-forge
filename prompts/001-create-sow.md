# Prompt 001 — Create or Revise Statement of Work

## Context

Stage 1 of Protocol 001 (Documentation-Driven Development). This prompt is used when starting a new feature, architectural change, or system revision.

## Prompt

You are creating or revising the Statement of Work (`docs/SOW.md`) for this project.

Given the following input:

**Feature/Change**: [describe the idea, decision, or change]

Produce a SOW (or SOW revision) that includes:

1. **Overview** — what the system is and what it does, in plain language
2. **Core Concepts** — the building blocks and their relationships
3. **Architectural Layers** — how components are organized
4. **Typical Flow** — end-to-end walkthrough of how things work
5. **Component Descriptions** — each component's role and responsibilities
6. **Data Model** — key data structures with examples (if applicable)
7. **Implementation Phases** — ordered phases with gates
8. **Key Differences** — comparison table vs prior version (if revising)
9. **Out of Scope** — what this SOW explicitly does not cover

Rules:
- Keep it concise and actionable (SOW, not a novel)
- Use concrete examples, not abstract descriptions
- Each phase must have a verifiable gate condition
- Reference relevant protocols

## Expected Output

A complete `docs/SOW.md` file ready for review.
