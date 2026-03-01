# Protocol 004 — Red-Green-Refactor Implementation

## Status

Active

## Purpose

Define the strict TDD cycle for all implementation work. No code is written without a failing test first. No refactoring without green tests. Every cycle produces a commit.

## Position in the Pipeline

```
SOW → Journey → Use Cases → Business Rules → Test Cases → Solution Draft → Implementation
                                                                                   ↑
                                                                            (this protocol)
```

This protocol governs Stage 7 of Protocol 001. It takes the red tests from Stage 5 (Protocol 002) and the architecture from Stage 6 (Protocol 003) and produces working code through disciplined TDD cycles.

## The Cycle

```
   ┌──────────┐
   │   RED    │  Write or select a failing test
   │          │  (test exists from Stage 5, or write new one for discovered requirements)
   └────┬─────┘
        │
        ▼
   ┌──────────┐
   │  GREEN   │  Write the MINIMUM code to make that test pass
   │          │  (no more, no less — ugly is fine)
   └────┬─────┘
        │
        ▼
   ┌──────────┐
   │ REFACTOR │  Clean up the code while keeping all tests green
   │          │  (rename, extract, simplify — but don't add behavior)
   └────┬─────┘
        │
        └──────→ Pick next failing test → repeat
```

## Rules

### 1. Red Before Green

You MUST NOT write implementation code without a failing test that demands it. If you discover a need for code that no test covers, write the test first.

The test suite from Stage 5 (Protocol 002) provides the initial red tests. As implementation progresses, you will discover additional tests needed — write them red, then make them green.

### 2. Minimum to Green

The GREEN step writes the **smallest, simplest code** that makes the failing test pass. Do not:
- Anticipate future tests
- Add error handling for untested scenarios
- Build abstractions before a second use case demands it
- Optimize before profiling

If it's ugly but green, that's correct. Refactor comes next.

### 3. Refactor Only When Green

Refactoring MUST happen only when all tests pass. Refactoring MUST NOT change behavior — tests stay green throughout. Refactoring includes:
- Rename variables/functions for clarity
- Extract shared logic into functions (only when duplication is real, not anticipated)
- Simplify conditionals
- Remove dead code

### 4. One Test at a Time

Work on one failing test at a time. Do not try to make multiple tests pass simultaneously. The discipline is:
1. Pick the next failing test
2. Run it — confirm it fails (RED)
3. Write code — run it — confirm it passes (GREEN)
4. Refactor — run all tests — confirm all still pass (REFACTOR)
5. Commit
6. Pick the next failing test

### 5. Commit at Each Color

Each color change produces a commit:

| Color | Commit Message Pattern | Example |
|-------|----------------------|---------|
| RED | `test(BR-NNN): add failing test for <scenario>` | `test(BR-001): add failing test for password too short` |
| GREEN | `feat(BR-NNN): implement <what>` | `feat(BR-001): implement password length validation` |
| REFACTOR | `refactor(BR-NNN): <what changed>` | `refactor(BR-001): extract validator to shared function` |

Stage 5 tests are already committed red. Implementation starts at GREEN.

### 6. Work Order: By BR, Bottom-Up

Implement BRs in dependency order. Start with foundational rules that other rules depend on. Within each phase, BRs can be implemented in any order.

### 7. Test Granularity

Each RED-GREEN-REFACTOR cycle targets **one test function**, not one test file. A BR test file may have 8 tests — that's 8 cycles (or more if refactoring between cycles adds value).

### 8. No Gold Plating

If a test passes, move on. Do not:
- Add features the tests don't require
- Handle errors the tests don't cover
- Build abstractions for hypothetical reuse
- Write documentation for internal functions

The tests are the specification. When they're green, you're done.

### 9. Discovered Requirements

During implementation, you will discover requirements not covered by existing tests. Handle them:

1. **Stop** the current GREEN step
2. **Write** a new test for the discovered requirement (RED)
3. **Commit** the new test
4. **Continue** — make the new test pass (GREEN)
5. **Resume** the original work

### 10. Integration Tests After Unit Tests

After all BR unit tests are green, write integration tests that verify cross-BR behavior. Integration tests follow the same RED-GREEN-REFACTOR cycle.

## Inputs

- Failing test suite from Stage 5 (`test/business-rules/BR-PROJ-NNN_*_test.<ext>`)
- Solution architecture from Stage 6 (`docs/solution/*.md`)
- Business rules (`docs/business-rules/BR-PROJ-NNN_*.md`)
- Use cases (`docs/use-cases/UC-PROJ-NNN_*.md`)

## Gate

All tests pass (green). Implementation matches:
- Solution draft interfaces
- Verification steps in use cases
- Traceability matrix — every BR has an enforcing component with passing tests

## Anti-Patterns

| Anti-Pattern | Why It's Wrong | Correct Approach |
|-------------|----------------|------------------|
| Write all code, then run tests | Defeats the purpose of TDD — you lose feedback | One test at a time |
| Make test pass, skip refactor | Technical debt accumulates | Always refactor when green |
| Refactor while red | No safety net — you don't know if refactor broke something | Only refactor when green |
| Write test after code | Tests confirm what you wrote, not what you need | Test first, always |
| Make 5 tests pass at once | Too much code without feedback — harder to debug | One test, one cycle |
| Add untested error handling | Dead code that may be wrong | Test the error case first |
