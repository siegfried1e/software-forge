# Software Forge

A documentation-driven development methodology that turns ideas into working software through a structured, repeatable pipeline.

## The Pipeline

```
SOW → User Journey → Use Cases → Business Rules → Test Cases → Solution Draft → Implementation
 │         │              │             │              │              │              │
 │         │              │             │              │              │              └─ Protocol 004
 │         │              │             │              │              └─ Protocol 003
 │         │              │             │              └─ Protocol 002
 │         │              │             │
 └─────────┴──────────────┴─────────────┴──────────────── Protocol 001 (stages 1-4)
```

Every feature follows this pipeline. No stage may be skipped. Each stage feeds the next.

## Protocols

Protocols define **how** to do each thing:

| # | Protocol | Governs |
|---|----------|---------|
| 001 | [Documentation-Driven Development](protocols/001-documentation-driven-development.md) | The full pipeline — stages 1 through 7 |
| 002 | [Test-Driven Business Rules](protocols/002-test-driven-business-rules.md) | Stage 5 — writing test cases from BRs |
| 003 | [Solution Architecture](protocols/003-solution-architecture.md) | Stage 6 — designing the technical solution |
| 004 | [Red-Green-Refactor](protocols/004-red-green-refactor.md) | Stage 7 — implementing with strict TDD |

## Prompts

Prompts are **executable instructions** — give them to an AI assistant or follow them yourself:

| # | Prompt | Pipeline Stage |
|---|--------|---------------|
| 001 | [Create SOW](prompts/001-create-sow.md) | Stage 1 |
| 002 | [Create User Journey](prompts/002-create-user-journey.md) | Stage 2 |
| 003 | [Create Use Cases](prompts/003-create-use-cases.md) | Stage 3 |
| 004 | [Create Business Rules](prompts/004-create-business-rules.md) | Stage 4 |
| 005 | [Create BR Test Cases](prompts/005-create-br-test-cases.md) | Stage 5 |
| 006 | [Create Solution Draft](prompts/006-create-solution-draft.md) | Stage 6 |
| 007 | [Implement (Red-Green-Refactor)](prompts/007-implement-red-green-refactor.md) | Stage 7 |

## How to Use

1. **Start a new feature**: Run Prompt 001 to create a Statement of Work
2. **Walk through the pipeline**: Run each subsequent prompt in order (002 → 007)
3. **Gate each stage**: Don't proceed until the current stage's gate conditions are met
4. **Adapt the templates**: Replace `PROJ` with your project identifier (e.g., `BR-MYAPP-001`)

The methodology is language-agnostic and framework-agnostic. The examples use generic placeholders — adapt the test file extensions, build commands, and tooling to your stack.

## Key Principles

- **No code without documentation** — every line traces back to a use case and business rule
- **Tests are the specification** — written before implementation, in a failing state
- **One test at a time** — strict Red-Green-Refactor discipline
- **Cross-references are mandatory** — use cases reference business rules and vice versa
- **Prompts drive execution** — reproducible, auditable stage transitions

## License

MIT
