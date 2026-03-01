# Prompt 004 — Create Business Rules from Use Cases

## Context

Stage 4 of Protocol 001 (Documentation-Driven Development). This prompt is used after use cases are written, to extract enforceable rules and cross-reference them back.

## Prompt

You are creating business rules based on use cases.

Given the following input:

**Use Cases**: [list of UC files to analyze, e.g., UC-PROJ-001 through UC-PROJ-005]

Perform these steps:

### Step 1: Extract Rules

Read each use case and identify enforceable rules. Look for:
- Validation constraints (what must be true for an action to proceed)
- Ordering constraints (what must happen before what)
- Behavioral requirements (how the system must behave)
- Security boundaries (what is forbidden)
- Interface requirements (what the user experience must include)

### Step 2: Write Business Rules

For each rule, create a file in `docs/business-rules/` following this template:

```markdown
# BR-PROJ-NNN: [Title]

## Identifier
BR-PROJ-NNN

## Title
[Short descriptive title]

## Domain
[Project Name] — [Subdomain]

## Normative Statement
[The rule, using MUST/SHOULD/MAY per RFC 2119]

## Scope
- **Covers**: [what this rule applies to]
- **Does not cover**: [explicit exclusions]

## Consequences
[What happens when the rule is violated]

## Valid Examples
| Input | Expected Result |
|-------|----------------|
| [valid scenario] | [expected outcome] |

## Invalid Examples
| Input | Expected Error |
|-------|---------------|
| [invalid scenario] | [error message referencing this BR] |

## References
- Related Protocols: [list]
- Related UCs: [list]
- Related BRs: [list]

## Status
Draft
```

### Step 3: Back-Reference

After creating all rules, update each use case's **Business Rules Applied** and **References** sections to reference the new BRs. Cross-references must be bidirectional:
- Each UC lists its applicable BRs
- Each BR lists the UCs it applies to

Rules:
- Number BRs sequentially from the last existing BR
- Every use case must reference at least one BR
- Every BR must be referenced by at least one UC
- Error messages in Invalid Examples must include the BR identifier (e.g., "BR-PROJ-001")
- Deduplicate — don't create a new BR if an existing one already covers the rule

## Expected Output

1. Numbered BR files in `docs/business-rules/`
2. Updated UC files with BR cross-references
