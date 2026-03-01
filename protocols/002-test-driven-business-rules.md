# Protocol 002 — Test-Driven Business Rules

## Status

Active

## Purpose

Define the standard for generating test cases from business rules. Every business rule MUST have a comprehensive test suite covering all valid examples, invalid examples, and edge cases — written before any implementation code. This is the bridge between documentation (Protocol 001 Stage 4) and implementation (Protocol 001 Stage 5).

## Position in the Pipeline

```
SOW → Journey → Use Cases → Business Rules → Test Cases → Implementation
                                                  ↑
                                            (this protocol)
```

This protocol adds Stage 4.5 between Business Rules and Implementation. Tests are written first (TDD). Implementation follows to make the tests pass.

## Rules

### 1. Every BR Gets a Test File

Each business rule `BR-PROJ-NNN` MUST have a corresponding test file:
```
test/business-rules/BR-PROJ-NNN_<ShortName>_test.<ext>
```

### 2. Minimum Test Coverage Per BR

Every BR MUST have at least:
- **One success test per valid example** — confirms the system allows what it should
- **One failure test per invalid example** — confirms the system rejects what it should, with the correct error message
- **One edge case test** — covers boundary conditions not explicitly listed in valid/invalid examples

### 3. Error Messages Are Part of the Contract

Tests MUST assert the exact error message text, including the BR identifier. Error messages are not implementation details — they are user-facing contracts defined in the BR.

Example:
```
assert error contains "BR-PROJ-001"
assert error contains "Password must be at least 8 characters"
```

### 4. Test Structure

Every test file follows this structure:

```
// TestBR_PROJ_NNN_<ShortName>_Success_<Scenario>
// TestBR_PROJ_NNN_<ShortName>_Failure_<Scenario>
// TestBR_PROJ_NNN_<ShortName>_Edge_<Scenario>
```

Use table-driven tests when multiple inputs map to the same assertion pattern.

### 5. Tests Are Red First

Tests MUST be written and committed in a failing state (red). They are the specification. Implementation makes them green. This sequence is enforced:

1. Write test file — commit (tests fail, this is expected)
2. Write implementation — commit (tests pass)
3. Refactor — commit (tests still pass)

### 6. Test Categories

Each BR test suite covers three categories:

| Category | Purpose | Naming |
|----------|---------|--------|
| **Success** | Valid inputs produce expected results | `Test..._Success_<Scenario>` |
| **Failure** | Invalid inputs produce correct error with BR identifier | `Test..._Failure_<Scenario>` |
| **Edge** | Boundary conditions, empty inputs, concurrent access, nil refs | `Test..._Edge_<Scenario>` |

### 7. What to Test Per BR Type

| BR Type | Test Focus |
|---------|-----------|
| **Validation** | Input accepted/rejected with correct error messages |
| **Communication** | Messages flow correctly through expected channels |
| **Configuration** | Default values applied correctly, overrides work |
| **Ordering** | Phase gates block premature progression, resume works |
| **Verification** | Health checks detect healthy/unhealthy components |
| **Scope** | Operations blocked/allowed per defined boundaries |

## Gate

All test files committed and failing (red). The test suite is the specification. No implementation code exists yet. Running the test suite shows all tests failing with clear assertion messages.
