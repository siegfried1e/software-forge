# Prompt 002 — Create User Journey

## Context

Stage 2 of Protocol 001 (Documentation-Driven Development). This prompt is used after the SOW is accepted, to describe how a human actually uses the system.

## Prompt

You are creating a user journey based on the current SOW (`docs/SOW.md`).

Given the following input:

**Journey Focus**: [describe what workflow or interaction this journey covers]

Produce a user journey in `docs/user-journeys/` that includes:

1. **Goal** — what the user achieves by the end of this journey
2. **Prerequisites** — what must exist before starting
3. **Steps** — numbered, concrete steps showing:
   - Real configuration or manifests the user would apply
   - Real CLI commands the user would run
   - Real system responses the user would see
   - Verification commands to confirm each step worked
4. **End State** — what the system looks like after completion
5. **Next Journey** — what the user would do next

Rules:
- Show, don't tell — concrete examples over abstract descriptions
- Every step must have a verification check
- Use the conversational interface where applicable (the user talks, the system responds)
- Number the journey file sequentially (check existing journeys first)

## Expected Output

A numbered journey file in `docs/user-journeys/` (e.g., `01-getting-started.md`).
