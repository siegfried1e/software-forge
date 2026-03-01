# Protocol 001 — Documentation-Driven Development

## Status

Active

## Purpose

Define the standard methodology for turning a high-level idea into implementation-ready specifications. Every feature or architectural change follows this pipeline before any code is written.

## The Pipeline

```
SOW → User Journey → Use Cases → Business Rules → Test Cases → Solution Draft → Implementation
```

Each stage feeds the next. No stage may be skipped.

### Stage 1: Statement of Work (SOW)

**Input**: A high-level idea, architectural decision, or feature request.

**Action**: Write or revise `docs/SOW.md` to capture:
- What the system is and what it does
- Core concepts and building blocks
- Architectural layers and component roles
- Typical flow (how things work end-to-end)
- Implementation phases with gates
- Key differences from prior versions (if revising)

**Output**: A single document that anyone can read to understand the system.

**Gate**: SOW is reviewed and accepted before proceeding.

### Stage 2: User Journey

**Input**: The SOW.

**Action**: Write user journeys in `docs/user-journeys/` that describe how a human actually uses the system step-by-step. Journeys are concrete, showing:
- Real manifests or configuration
- Real CLI commands
- Real system responses
- Verification steps

**Output**: Numbered journey files (e.g., `01-getting-started.md`).

**Gate**: Journeys cover the primary workflows described in the SOW.

### Stage 3: Use Cases

**Input**: User journeys.

**Action**: Formalize each journey step into use cases in `docs/use-cases/`. Each use case follows the standard template:
- Identifier, Title, Domain, Summary
- Actors, Preconditions
- Main Flow (with actual dialogue for conversational interfaces)
- Alternative Flows
- Error Flows
- Postconditions
- Business Rules Applied (back-referenced after Stage 4)
- References

**Output**: Numbered UC files (e.g., `UC-PROJ-001_UserRegistration.md`).

**Gate**: Every journey step has at least one use case. Every use case has alternative and error flows.

### Stage 4: Business Rules

**Input**: Use cases.

**Action**: Extract enforceable rules from use cases and write them in `docs/business-rules/`. Each rule follows the standard template:
- Identifier, Title, Domain
- Normative Statement (MUST/SHOULD/MAY per RFC 2119)
- Scope (what it covers and does not cover)
- Consequences (what happens on violation)
- Valid and Invalid Examples
- References

After creating rules, back-reference them in the use cases that triggered them.

**Output**: Numbered BR files (e.g., `BR-PROJ-001_PasswordComplexity.md`).

**Gate**: Every use case references at least one business rule. Every business rule is referenced by at least one use case. Cross-references are bidirectional.

### Stage 5: Test Cases (TDD)

**Input**: Business rules.

**Action**: For each BR, generate a test file in `test/business-rules/` covering:
- One success test per valid example
- One failure test per invalid example (asserting exact error message + BR identifier)
- At least one edge case test per BR

Tests are committed in a failing state (red). They are the specification. See Protocol 002 for full rules.

**Output**: Test files at `test/business-rules/BR-PROJ-NNN_<ShortName>_test.<ext>`.

**Gate**: All tests compile. All tests fail (red). Every BR has at least one success, one failure, and one edge case test.

### Stage 6: Solution Architecture Draft

**Input**: Test cases + business rules + technology stack.

**Action**: Design the technical solution in `docs/solution/`. Define components, interfaces, data flow, error handling, build/deploy, and a traceability matrix mapping every BR to its enforcing component. See Protocol 003 for full rules.

**Output**: Files in `docs/solution/`.

**Gate**: All components have defined interfaces. All BRs mapped to components. Solution reviewed and accepted.

### Stage 7: Implementation (Red-Green-Refactor)

**Input**: Solution draft + test cases + use cases + business rules.

**Action**: Implement code using strict Red-Green-Refactor TDD cycles (Protocol 004):
1. **RED** — Pick a failing test, confirm it fails
2. **GREEN** — Write the minimum code to make it pass
3. **REFACTOR** — Clean up while keeping all tests green
4. **Commit** — One commit per color change
5. **Repeat** — Next failing test

Work through BRs in dependency order (foundational validation first, integration last). One test at a time. No code without a failing test. No refactoring while red.

**Output**: Working code in the packages defined by the solution draft.

**Gate**: All tests pass (green). Implementation matches the solution draft interfaces and the verification steps in the use cases. Every commit follows the RED-GREEN-REFACTOR pattern.

## Rules

1. **No code without documentation.** Every implementation MUST trace back to a use case and business rule.
2. **Stages are sequential.** You MUST complete each stage's gate before starting the next.
3. **Changes flow forward.** If the SOW changes, user journeys, use cases, and business rules MUST be reviewed and updated.
4. **Cross-references are mandatory.** Use cases reference business rules. Business rules reference use cases. Both reference protocols.
5. **Prompts drive execution.** Each stage transition SHOULD be captured as a prompt in `prompts/` so the work is reproducible and auditable.

## Prompt Commands

Each stage has a corresponding prompt in `prompts/` that can be used to execute it:

| Stage | Prompt | Purpose |
|-------|--------|---------|
| 1. SOW | `prompts/001-create-sow.md` | Create or revise the Statement of Work |
| 2. Journey | `prompts/002-create-user-journey.md` | Create a user journey from the SOW |
| 3. Use Cases | `prompts/003-create-use-cases.md` | Formalize journey steps into use cases |
| 4. Business Rules | `prompts/004-create-business-rules.md` | Extract rules from use cases and cross-reference |
| 5. Test Cases | `prompts/005-create-br-test-cases.md` | Generate TDD test suite from business rules |
| 6. Solution Draft | `prompts/006-create-solution-draft.md` | Design technical solution with interfaces and traceability |
| 7. Implementation | `prompts/007-implement-red-green-refactor.md` | Implement code via strict Red-Green-Refactor TDD cycles (Protocol 004) |

These prompts are reusable. Each time a new feature is started, run them in sequence.
