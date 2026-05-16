# Architectural Decision Record — Template

Copy this template to `docs/architecture/<NNN>-<kebab-title>.md` or append it as an entry in `docs/coordination/decision-log.md`.

---

### ADR-<NNN> — <Title>

**Date**: YYYY-MM-DD
**Status**: Proposed | Accepted | Superseded | Retroactive
**Author**: <agent role or person>

#### Context

What situation, constraint, or question prompted this decision? Include the project state at the time and why the decision could not be deferred.

#### Decision

State the decision clearly in one or two sentences. Use MUST/SHOULD/MAY (RFC 2119) where precision matters.

#### Rationale

Why this option was chosen over alternatives. List the alternatives considered and the reason each was not chosen.

#### Consequences

What becomes easier, harder, or required as a result of this decision? List downstream artifacts (BRs, charts, solution docs, manifests) that depend on or are constrained by this decision.

#### Cross-References

- Pipeline artifacts constrained by this decision: (BR-PROJ-NNN, UC-PROJ-NNN, solution doc section, etc.)
- Related ADRs: (ADR-NNN)
- Supersedes / superseded by: (if applicable)

#### Retroactive Note *(include only when Status = Retroactive)*

When was this decision first applied without documentation? What triggered the retroactive entry? Reference the formalization dispatch (D-NNN) or the phase gate where the gap was discovered.
