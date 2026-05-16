# Prompt 005 — Create Test Specifications from Business Rules

## Context

Stage 5 of Protocol 001 (Documentation-Driven Development), governed by Protocol 002 (Test-Driven Business Rules). This prompt is used after business rules are written, to design comprehensive test specifications before any code is written.

## Prompt

You are designing test specifications based on business rules.

Given the following input:

**Business Rules**: [list of BR files to cover, e.g., BR-PROJ-001 through BR-PROJ-010]

For each business rule, perform these steps:

### Step 1: Analyze the BR

Read the business rule and extract:
- Every **valid example** → becomes a success scenario
- Every **invalid example** → becomes a failure scenario with exact error message
- **Edge cases** not explicitly listed but implied by the rule's scope

### Step 2: Write Test Specification

Create a test specification at `docs/test-specifications/BR-PROJ-NNN_<ShortName>_test-spec.md` with:

#### Success Scenarios
One scenario per valid example from the BR. Define the concrete input and expected result.

```markdown
### S-001: BR_PROJ_NNN_ShortName_Success_ValidScenario

**Category**: Success
**Input**: <concrete input values>
**Expected Outcome**: <expected result — what the system produces or allows>
**BR Reference**: BR-PROJ-NNN
```

#### Failure Scenarios
One scenario per invalid example from the BR. Define the concrete input and exact error message:

```markdown
### S-002: BR_PROJ_NNN_ShortName_Failure_InvalidScenario

**Category**: Failure
**Input**: <concrete input values>
**Expected Outcome**: Error — "<exact error message> (BR-PROJ-NNN)"
**BR Reference**: BR-PROJ-NNN
```

#### Edge Case Scenarios
At least one edge case per BR covering boundary conditions:
- Empty/nil inputs
- Maximum values
- Concurrent access (if applicable)
- State transitions at boundaries

```markdown
### S-003: BR_PROJ_NNN_ShortName_Edge_BoundaryCondition

**Category**: Edge
**Input**: <concrete input at boundary>
**Expected Outcome**: <expected behavior at the boundary>
**BR Reference**: BR-PROJ-NNN
**Notes**: <why this boundary matters>
```

### Step 3: Document Coverage

At the top of each test specification file, add a coverage summary listing:
- The BR being specified
- Number of success scenarios
- Number of failure scenarios
- Number of edge case scenarios
- Any scenarios explicitly out of scope (from the BR's "Does not cover")

## Rules

- Test specifications are design documents — do NOT write code
- Error messages in failure scenarios MUST match the BR's Invalid Examples exactly and include the BR identifier
- Each scenario name follows: `BR_PROJ_NNN_ShortName_{Success|Failure|Edge}_Scenario`
- Specification files go in `docs/test-specifications/`
- Every scenario MUST have concrete input values and a concrete expected outcome

## Expected Output

For each BR:
1. A specification file at `docs/test-specifications/BR-PROJ-NNN_<ShortName>_test-spec.md`
2. Coverage summary at the top of each file
3. No code — only scenario definitions

## Checklist

After generating all specification files, verify:
- [ ] Every BR has a test specification file
- [ ] Every valid example in every BR has a success scenario
- [ ] Every invalid example in every BR has a failure scenario with exact error message
- [ ] Every BR has at least one edge case scenario
- [ ] Every scenario has concrete input and expected outcome
- [ ] Error messages in scenarios include BR identifiers
- [ ] No code was written — specifications only
