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

### 8. Runtime Behavior Coverage

Every BR whose normative statement specifies *runtime behavior of a component* — provider implementations, message claimers/publishers, reconcilers, controllers, subprocess invocation, network calls to external services — MUST have an accompanying test specification that includes at least one **end-to-end integration scenario** that exercises the actual runtime path described.

An end-to-end scenario specification MUST:

1. **Identify the runtime entry point** (e.g., "operator reconcile loop is invoked", "worker claims a queue message", "BFF receives an HTTP POST").
2. **Identify the runtime exit point** (e.g., "CR reaches terminal phase", "completion message published to the bus", "client receives HTTP response with body").
3. **Specify the cross-component path** between entry and exit (e.g., "operator → CR → worker → provider → external API → response → completion publish").
4. **Specify the expected observable signal** (CRD field value, message subject + payload, HTTP response code + body, log line, metric increment).
5. **NOT delegate the runtime path to another component's scope** — if the BR specifies that behavior X happens, the test spec for that BR MUST include a scenario that verifies X happens end-to-end in the runtime path the BR describes.

The Stage 5 gate CANNOT mark PASS if the BR specifies runtime behavior AND the test spec does not contain at least one such scenario.

**Scope clarifications**:

- This rule does NOT require test specs to dictate implementation language or test framework. The scenario specification describes *what to observe*, not *how to execute*.
- This rule does NOT apply to BRs that purely specify static contracts (CRD schema, configuration values, naming conventions). Those legitimately have no runtime path.
- If a BR's runtime path crosses multiple components, the test spec MUST still include the end-to-end scenario. The Test Architect drafts what to observe; the Developer at Stage 7 chooses the implementation approach (mocked component boundaries with real runtime under test, real integration test, etc.).
- Test specs MAY mark end-to-end scenarios with explicit `[INTEGRATION]` or `[E2E]` tags to distinguish them from unit-level scenarios. A canonical scenario shape is provided in `templates/test-spec-with-e2e.md`.

**Reject pattern** (Stage 5 gate violation): test spec includes ONLY scenarios that mock the underlying runtime (subprocess mocks, HTTP mocks, fake CRD stores) when the BR specifies behavior that depends on real component interaction.

**Accept pattern**: test spec includes both unit-level scenarios (mocked dependencies, fast feedback) AND at least one end-to-end scenario (real cross-component path, observable runtime signal).

**Why this rule exists**: when a BR specifies runtime behavior but the test spec covers only unit-level scenarios, the runtime path is never exercised until integration testing. Defects in cross-component interaction surface late, when the cost of fixing them is highest (specifier amendment + test architect amendment + solution architect amendment + N developer dispatches + re-integration). Requiring an end-to-end scenario at Stage 5 surfaces the constraint at design time. This rule applies the same "make the implicit explicit" principle as Protocol 001's Architectural Decision Records and Protocol 003's Cross-Component Service References.

## Gate

All test specification files exist in `docs/test-specifications/`. Every BR has a specification. Every specification has at least one success, one failure, and one edge case scenario. Every BR specifying runtime behavior has at least one end-to-end integration scenario (Rule 8). Every scenario has concrete input, expected outcome, and BR reference. No code has been written — specifications are design documents only.
