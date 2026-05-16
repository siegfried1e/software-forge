# Prompt 007 — Implement with Red-Green-Refactor

## Purpose

Execute Stage 7 of Protocol 001. Take the test specifications from Stage 5, the solution architecture from Stage 6, and produce test code and working implementation through strict TDD cycles.

## Prerequisites

- All test specifications exist in `docs/test-specifications/` (Stage 5 complete)
- Solution draft exists in `docs/solution/` (Stage 6 complete)
- Business rules exist in `docs/business-rules/` (Stage 4 complete)

## Instructions

You are implementing the system using strict Red-Green-Refactor TDD (Protocol 004). For each BR, you write the test code from the test specification, then implement the code to make it pass.

### Step 1: Inventory

List all test specification scenarios grouped by BR. Determine the implementation order:

1. Read all test specification files in `docs/test-specifications/`
2. Read the solution architecture in `docs/solution/`
3. Read the traceability matrix in `docs/solution/traceability.md`
4. Produce a work plan: which BR to implement first, which scenario within that BR to start with

### Step 2: Pick a BR

Select the next BR from the dependency order. Read:
- The BR document (`docs/business-rules/BR-PROJ-NNN_*.md`)
- The test specification (`docs/test-specifications/BR-PROJ-NNN_*_test-spec.md`)
- The solution component that enforces it (`docs/solution/traceability.md`)
- The relevant solution design file (`docs/solution/*.md`)

### Step 3: RED — Write the Test

1. Pick the first scenario from the test specification
2. Translate it into test code in `test/business-rules/BR-PROJ-NNN_<ShortName>_test.<ext>`
3. Run the test and confirm it fails with a clear assertion message
4. If it doesn't compile, create the minimum stub (empty function, type definition) to make it compile — but still fail

### Step 4: GREEN — Write Minimum Code

1. Read the solution draft for the target component
2. Write the **minimum code** to make this one test pass
3. Place code in the package the test imports (as defined in the solution draft)
4. Run the test and confirm it passes
5. Run **all** tests and confirm no regressions

### Step 5: REFACTOR

1. Review the code you just wrote
2. Clean up: rename, extract, simplify — without changing behavior
3. Run all tests and confirm all still pass
4. If nothing to refactor, skip this step

### Step 6: Commit

Commit with the appropriate message:
- If you wrote the test: `test(BR-NNN): add failing test for <scenario>`
- If you made a test pass: `feat(BR-NNN): implement <what>`
- If you refactored: `refactor(BR-NNN): <what changed>`

### Step 7: Repeat

Go back to Step 3 with the next scenario from the test specification for the same BR. When all scenarios for a BR are implemented and green, go back to Step 2 with the next BR.

### Step 8: Discovered Requirements

If during GREEN you discover a requirement not covered by existing test specifications:
1. Stop the current GREEN work
2. Write a new test for the discovered requirement
3. Run it — confirm it fails (RED)
4. Commit: `test(BR-NNN): add test for <discovered requirement>`
5. Make it pass (GREEN)
6. Commit: `feat(BR-NNN): implement <discovered requirement>`
7. Resume the original GREEN work

### Step 9: Integration

After all BR unit tests are green:
1. Write integration tests that verify cross-BR behavior
2. Follow the same RED-GREEN-REFACTOR cycle
3. Place integration tests in `test/integration/`

## Constraints

- **One test at a time.** Do not try to make multiple tests pass simultaneously.
- **Minimum code only.** Do not anticipate future tests. Ugly but green is correct.
- **Refactor only when green.** Never change code structure while tests are failing.
- **No gold plating.** If the test passes, move on. Don't add untested features.
- **Follow the solution draft.** Code structure, module names, and interfaces must match `docs/solution/*.md`.
- **Follow Protocol 004.** This prompt executes that protocol.
- **Trace to specifications.** Every test MUST correspond to a scenario in the test specification.

## Output

At the end of this prompt's execution:
- All tests in `test/business-rules/` pass (green)
- Every test traces back to a scenario in `docs/test-specifications/`
- Code is organized per `docs/solution/` directory structure
- Every commit follows the RED-GREEN-REFACTOR pattern
- No untested code exists
