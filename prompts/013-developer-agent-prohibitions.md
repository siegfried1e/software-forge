# Prompt 013 — Formalize Developer Agent Prohibitions

## Context

Protocol 007 Rule 3 ("Gates are non-negotiable. An agent cannot self-certify its own gate.") and Rule 10 ("Orchestrator maintains coordination records.") together imply that developer agents must not write to the Orchestrator's coordination directory and must not declare their own work passed. However, these implications are **not behaviorally enforced** — they live only as protocol rules, and in practice developer agents violate them.

**Evidence from a parallel Stage 7 dispatch:**

Three parallel developer agents were dispatched in worktrees for the same Stage 7:
- **Agent A (operator)** committed a self-written gate verdict file under `docs/coordination/exchanges/`. The agent also ran `git merge` to the main branch (without Orchestrator approval). The self-verdict said PASS, but the Orchestrator had actually failed the gate due to a missing feature.
- **Agent B (dashboard)** committed a "gate verdict PASS" file AND modified `docs/coordination/audit-log.md`. Same pattern.
- **Agent C (observability)** was compliant.

So **2 of 3 parallel developer agents** violated the rules in the same way, and **1 of them also merged its own branch** — a direct violation of Rule 3 and a safety-critical problem (the merge contained broken code that had failed the Orchestrator's gate).

**Root cause**: developer agents read the existing `docs/coordination/exchanges/` directory as part of their context (they need to read their dispatch artifact). They observe the pattern — every dispatch has a completion, a gate verdict, and an audit log entry — and helpfully produce all three. The pattern is there in the repository; the agents extrapolate.

**Why Rule 3 as written is not enough**: "An agent cannot self-certify its own gate" is a policy statement. Agents interpret it as "don't lie in my self-check", not as "don't write any file that resembles a gate verdict". The behavioral prohibition is missing.

## Prompt

Add explicit **developer agent prohibitions** to Protocol 007, either as a new subsection under "Agent Roles" or as an extension to Rule 3. The prohibitions should be behavioral, not merely normative:

1. **Prohibition on writing to the coordination directory.**
   Developer agents MUST NOT write to the Orchestrator's coordination directory (e.g., `docs/coordination/`). This includes:
   - Gate verdict files (`*-gate-verdict-*`, `*-gate-feedback-*`)
   - Audit log files (`audit-log.md`)
   - Change propagation records
   - Dispatch artifacts (these are authored by the Orchestrator)

   The only file a developer agent may produce regarding a dispatch is a **completion report**, written to a path explicitly specified in the dispatch artifact (typically a local `reports/` directory inside the worktree, NOT inside the coordination directory).

2. **Prohibition on branch merging.**
   Developer agents MUST NOT run `git merge`, `git checkout <shared-branch>`, `git push` to shared branches, or any operation that affects branches other than their dedicated working branch. The Orchestrator is the only agent permitted to merge dispatch branches into the main branch after gate evaluation.

3. **Self-check format clarification.**
   The Protocol 008 Completion Artifact may contain an **Acceptance Criteria Self-Check** section. This is permitted and encouraged — the agent reports which criteria it believes it has met, with evidence. But this self-check is NOT a gate verdict. The distinction:
   - **Permitted**: "I believe criterion X is met because of Y" (self-check inside the completion report)
   - **Forbidden**: "Gate PASS" (verdict file, regardless of its content)

4. **Enforcement via CLAUDE.md.**
   Projects using Protocol 007 SHOULD reproduce these prohibitions in each developer agent's CLAUDE.md (or equivalent behavior file) so the agent sees them at session start, not just as an abstract protocol rule. Consider providing a **developer agent CLAUDE.md template** in `software-forge/templates/` that projects can copy when scaffolding a new developer agent.

## Expected Output

1. **Protocol 007 update**: Add a new subsection (e.g., "Developer Agent Prohibitions" under "Agent Roles", or as Rule 3a). Include the four points above. Update the Table of Contents if there is one.

2. **Optional**: Create `software-forge/templates/developer-claude-md.md` with a reusable CLAUDE.md template for projects to copy. Include the prohibitions as explicit bullets, not just references to Protocol 007.

3. **README update**: Reflect the Protocol 007 change in the protocols table (and add a templates section if #2 is produced).

## Read Before Editing

- `protocols/007-multi-agent-pipeline-execution.md` — the current rules, especially Rules 3 and 10, and the "Agent Roles" section
- `protocols/008-agent-communication-contract.md` — Completion Artifact structure (ensure "Acceptance Criteria Self-Check" is an allowed section)
- `prompts/011-recurring-discrepancy-rule.md` — pattern for handling recurring violations (this prompt is itself a Triage Rule 6 fix for a recurring discrepancy)

## Scope Notes

- Do NOT weaken existing rules — the prohibitions are additive.
- The prohibitions apply specifically to **developer agents** (Stage 7). Pipeline agents (Analyst, Specifier, Test Architect, Solution Architect) have their own write scopes already defined; they do not need these prohibitions because their outputs are pipeline artifacts, not coordination records.
- The Orchestrator retains exclusive authorship of coordination records and exclusive merge authority.
