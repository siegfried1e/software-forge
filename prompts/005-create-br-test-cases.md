# Prompt 005 — Create Test Cases from Business Rules

## Context

Stage 4.5 of Protocol 001 (Documentation-Driven Development), governed by Protocol 002 (Test-Driven Business Rules). This prompt is used after business rules are written, to generate a comprehensive test suite before any implementation.

## Prompt

You are creating TDD test cases based on business rules.

Given the following input:

**Business Rules**: [list of BR files to cover, e.g., BR-PROJ-001 through BR-PROJ-010]

For each business rule, perform these steps:

### Step 1: Analyze the BR

Read the business rule and extract:
- Every **valid example** → becomes a success test
- Every **invalid example** → becomes a failure test with exact error message
- **Edge cases** not explicitly listed but implied by the rule's scope

### Step 2: Write Test File

Create a test file at `test/business-rules/BR-PROJ-NNN_<ShortName>_test.<ext>` with:

#### Success Tests
One test per valid example from the BR. The test constructs a valid input and asserts it is accepted.

```
func TestBR_PROJ_NNN_ShortName_Success_ValidScenario(t *testing.T) {
    // Arrange: construct valid input per BR valid example
    // Act: call the validation/enforcement function
    // Assert: no error, expected result produced
}
```

#### Failure Tests
One test per invalid example from the BR. The test constructs an invalid input and asserts:
1. It is rejected
2. The error message contains the BR identifier (`BR-PROJ-NNN`)
3. The error message contains the human-readable reason

```
func TestBR_PROJ_NNN_ShortName_Failure_InvalidScenario(t *testing.T) {
    // Arrange: construct invalid input per BR invalid example
    // Act: call the validation/enforcement function
    // Assert: error returned
    // Assert: error contains "BR-PROJ-NNN"
    // Assert: error contains descriptive message
}
```

#### Edge Case Tests
At least one edge case per BR covering boundary conditions:
- Empty/nil inputs
- Maximum values
- Concurrent access (if applicable)
- State transitions at boundaries

```
func TestBR_PROJ_NNN_ShortName_Edge_BoundaryCondition(t *testing.T) {
    // Arrange: construct edge case input
    // Act: call the validation/enforcement function
    // Assert: correct behavior at boundary
}
```

### Step 3: Use Table-Driven Tests Where Appropriate

When a BR has multiple valid or invalid examples following the same pattern, use table-driven tests.

### Step 4: Document Test Coverage

At the top of each test file, add a comment block listing:
- The BR being tested
- Number of success tests
- Number of failure tests
- Number of edge case tests
- Any scenarios explicitly out of scope (from the BR's "Does not cover")

## Rules

- Tests MUST compile but MUST fail (red) — they are the specification, not the implementation
- Error messages in assertions MUST match the BR's Invalid Examples exactly
- Each test function name follows: `TestBR_PROJ_NNN_ShortName_{Success|Failure|Edge}_Scenario`
- Test files go in `test/business-rules/`
- Do NOT write implementation code — only tests

## Expected Output

For each BR:
1. A test file at `test/business-rules/BR-PROJ-NNN_<ShortName>_test.<ext>`
2. All tests compile
3. All tests fail (red) — this is correct and expected
4. Coverage comment block at the top of each file

## Checklist

After generating all test files, verify:
- [ ] Every BR has a test file
- [ ] Every valid example in every BR has a success test
- [ ] Every invalid example in every BR has a failure test with error message assertion
- [ ] Every BR has at least one edge case test
- [ ] All test files compile
- [ ] All tests fail (red)
- [ ] Error messages in tests include BR identifiers
