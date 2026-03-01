# Prompt 003 — Create Use Cases from User Journey

## Context

Stage 3 of Protocol 001 (Documentation-Driven Development). This prompt is used after a user journey is written, to formalize each step into structured use cases.

## Prompt

You are creating use cases based on a user journey.

Given the following input:

**User Journey**: [path to the journey file, e.g., `docs/user-journeys/01-getting-started.md`]
**Interface Type**: [conversational / CLI / API / mixed]

For each distinct interaction in the journey, produce a use case in `docs/use-cases/` following this template:

```markdown
# UC-PROJ-NNN: [Title]

## Identifier
UC-PROJ-NNN

## Title
[Short descriptive title]

## Domain
[Project Name] — [Subdomain]

## Summary
[2-3 sentence overview]

## Actors
- **Primary**: [who initiates]
- **Secondary**: [who participates]

## Preconditions
- [required state/resources]

## Main Flow
[Numbered steps. For conversational interfaces, show actual dialogue
between User and System using blockquotes.]

## Alternative Flows
### AF-1: [Scenario name]
[Steps]

## Error Flows
### EF-1: [Error scenario]
[Steps with system response and suggested fix]

## Postconditions
- [final state after success]

## Business Rules Applied
- [leave placeholder — filled in after Stage 4]

## References
- Related Protocols: [list]
- Related UCs: [list]

## Status
Draft
```

Rules:
- Each use case is self-contained but references related UCs
- For conversational interfaces, main flows MUST show actual dialogue (User says X, System responds Y)
- Every UC must have at least one alternative flow and one error flow
- Error flows must include the system's diagnostic message and suggested fix
- Number UCs sequentially from the last existing UC
- Leave Business Rules Applied as placeholder — they get filled in after BR creation (Stage 4)

## Expected Output

One or more UC files in `docs/use-cases/`, numbered sequentially.
