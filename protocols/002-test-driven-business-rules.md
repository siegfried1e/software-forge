# Protocol 002 — Test-Driven Business Rules

## Status

Active

## Purpose

Define the standard for designing test specifications from business rules. Every business rule MUST have a comprehensive test specification covering all valid examples, invalid examples, and edge cases — designed before any code is written. Test specifications are design documents that define what to test, not how. This is the bridge between documentation (Protocol 001 Stage 4) and implementation (Protocol 001 Stage 7).

## Position in the Pipeline

```
SOW → Journey → Use Cases → Business Rules → Test Specifications → Solution Draft → Implementation
                                                      ↑
                                                (this protocol)
```

This protocol governs Stage 5 of Protocol 001. Test specifications are designed first. Implementation (Stage 7) writes the test code and makes it pass.

## Rules

### 1. Every BR Gets a Test Specification

Each business rule `BR-PROJ-NNN` MUST have a corresponding test specification file:
```
docs/test-specifications/BR-PROJ-NNN_<ShortName>_test-spec.md
```

### 2. Minimum Scenario Coverage Per BR

Every BR MUST have at least:
- **One success scenario per valid example** — describes what the system should accept and the expected result
- **One failure scenario per invalid example** — describes what the system should reject and the exact error message
- **One edge case scenario** — covers boundary conditions not explicitly listed in valid/invalid examples

### 3. Error Messages Are Part of the Contract

Test specifications MUST include the exact error message text, including the BR identifier. Error messages are not implementation details — they are user-facing contracts defined in the BR.

Example scenario:
```
Scenario: Password too short
Category: Failure
Input: password = "abc"
Expected Error: "Password must be at least 8 characters (BR-PROJ-001)"
BR Reference: BR-PROJ-001
```

### 4. Specification Structure

Every test specification file follows this structure:

```markdown
# Test Specification — BR-PROJ-NNN: <Title>

## BR Reference
BR-PROJ-NNN

## Coverage Summary
- Success scenarios: <count>
- Failure scenarios: <count>
- Edge case scenarios: <count>
- Out of scope: <list from BR's "Does not cover">

## Scenarios

### S-001: <Scenario Name>

**Category**: Success | Failure | Edge
**Input**: <concrete input values>
**Expected Outcome**: <expected result for success, expected error for failure>
**BR Reference**: BR-PROJ-NNN
**Notes**: <optional — any context the Developer needs when writing the test>
```

### 5. Scenario Naming

Each scenario follows the naming convention: `S-NNN: BR_PROJ_NNN_ShortName_{Success|Failure|Edge}_Scenario`. This name maps directly to the test function name the Developer will write in Stage 7.

### 6. Scenario Categories

| Category | Purpose | Required Content |
|----------|---------|-----------------|
| **Success** | Valid inputs produce expected results | Concrete input, expected result |
| **Failure** | Invalid inputs produce correct error with BR identifier | Concrete input, exact error message including BR identifier |
| **Edge** | Boundary conditions, empty inputs, concurrent access, nil refs | Concrete input, expected behavior at the boundary |

### 7. What to Specify Per BR Type

| BR Type | Specification Focus |
|---------|-------------------|
| **Validation** | Input accepted/rejected with expected error messages |
| **Communication** | Messages flow correctly through expected channels |
| **Configuration** | Default values applied correctly, overrides work |
| **Ordering** | Phase gates block premature progression, resume works |
| **Verification** | Health checks detect healthy/unhealthy components |
| **Scope** | Operations blocked/allowed per defined boundaries |

## Gate

All test specification files exist in `docs/test-specifications/`. Every BR has a specification. Every specification has at least one success, one failure, and one edge case scenario. Every scenario has concrete input, expected outcome, and BR reference. No code has been written — specifications are design documents only.
