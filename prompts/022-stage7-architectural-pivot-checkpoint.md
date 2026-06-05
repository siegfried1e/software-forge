# Prompt 022 — Stage 7 gate: Pre-Flight Architectural-Pivot Checkpoint MUST block tactical dispatches that expand architectural surface area

> **Source**: drafted by Mila Orchestrator (`/home/themaster/workspace/agentic`) on 2026-06-04 after a forensic audit of a remediation chain that propagated an undocumented substrate change through three Stage 7 dispatches before detection.
> **To submit**: copy this file to `/home/themaster/workspace/evocatio/software-forge/prompts/022-stage7-architectural-pivot-checkpoint.md`, review/refine, and integrate into Protocol 001 (Stage 7 gate criteria) and Protocol 007 (Orchestrator Sequencing rules).

## Context

A project running the full Protocol 001 + Protocol 007 pipeline can still ship architectural changes that bypass every upstream stage — provided those changes are framed as "tactical Stage 7 fixes" rather than as the architectural decisions they actually are. The methodology has no explicit pre-flight check that catches scope-expanding dispatches at authoring time, so the Orchestrator can dispatch a Stage-4-authorized "1-line fix" whose acceptance criteria silently introduce:

- a new Kubernetes resource kind (Secret, ConfigMap, Role, NetworkPolicy, Job, CronJob, ServiceAccount, CRD) the project did not previously create;
- a substrate change (hostPath ↔ PVC ↔ Secret/ConfigMap, in-memory ↔ persistent, single-cluster ↔ external service) that alters the trust boundary or multi-node story of an existing data flow;
- a new HTTP / gRPC / messaging contract surface (new route, new subject, new RPC, new event topic) for which no Contract Registry entry exists;
- a new RBAC binding or cross-namespace access pattern for which no Cross-Component Service References entry (Prompt 020) exists;
- a citation to an ADR or BR that contradicts the dispatch's actual design (the loudest stop-and-question signal, often the easiest to ignore under deadline pressure).

All five signals are present and visible at dispatch authoring time. The Orchestrator has the context to detect each one. What the methodology lacks is the explicit instruction to **stop and check**, framed as a gate that fires before Stage 7 dispatch — not after Stage 7 merge.

The cost of detection-after-merge vs detection-at-pre-flight:

- **Pre-flight**: a 60-second check (5 questions) catches the scope expansion at dispatch authoring time. The Orchestrator halts the Stage 7 dispatch and dispatches the missing upstream stages (Stage 4 BR amendment, Stage 6 Solution + Contract Registry entry, ADR amendment) first. No merged code is wrong; no downstream cascade.
- **Post-merge**: a forensic audit must reconstruct what was decided where, then dispatch retroactive Stage 1-6 work to formalize the architecture that already shipped. Working software stays (per existing CLAUDE.md guidance), but the methodology has to absorb the rework. Each undocumented architectural choice that survives N stages costs N rounds of retroactive Specifier / Test Architect / Solution Architect / Analyst dispatches to bring the documentation back in sync with the code.

The detection-gap pattern is structurally identical to the ones Prompts 019, 020, 021 address — implicit decisions that survive past the gate that should have caught them — but it sits at a different gate (Stage 7 pre-flight) and triggers on different signals (architectural surface expansion, not undocumented decisions or untested runtime paths).

## Prompt

Add the following gate criterion to Protocol 001 (Documentation-Driven Development), Stage 7 gate evaluation, AND a sibling rule to Protocol 007 (Multi-Agent Pipeline Execution) Orchestrator Sequencing:

> ### Stage 7 Pre-Flight Architectural-Pivot Checkpoint (added 202X)
>
> Before any Stage 7 Developer dispatch ships, the Orchestrator MUST evaluate all five questions and confirm that each answer is either "No" or "Yes — already covered upstream by [specific BR / Solution / ADR citation]". If any answer is "Yes" without upstream coverage, the Orchestrator MUST halt the Stage 7 dispatch and dispatch the missing upstream stage(s) first.
>
> 1. **New K8s (or platform) resource kind**: does this dispatch's acceptance criteria CREATE a kind of resource that did not exist in the project's deployment surface before — Secret, ConfigMap, Role, ClusterRole, NetworkPolicy, Job, CronJob, ServiceAccount, CRD, Webhook, mutating/validating admission policy, custom resource? If yes → architectural; require Stage 4 BR + Stage 6 Solution + ADR before Stage 7.
>
> 2. **Storage / state substrate change**: does this dispatch CHANGE the substrate of an existing data flow — hostPath ↔ PVC, hostPath ↔ Secret/ConfigMap, in-memory ↔ persistent, single-process ↔ multi-process, single-cluster ↔ external managed service, sync ↔ async/queued? Substrate changes alter trust boundary, multi-node story, durability story, and management UX. If yes → architectural; require ADR + BR + Solution.
>
> 3. **New contract surface**: does this dispatch INTRODUCE a new HTTP route, gRPC method, NATS / Kafka / RabbitMQ subject, event topic, webhook callback, or any other inter-process contract surface? If yes → require Contract Registry entry (Solution Architect dispatch) before Stage 7.
>
> 4. **New RBAC binding or cross-namespace access**: does this dispatch ADD a new ServiceAccount, Role/ClusterRole, RoleBinding/ClusterRoleBinding, cross-namespace Service reference, or otherwise expand the principal-to-resource access matrix? If yes → require Stage 6 Cross-Component Service References update (per Prompt 020) + Solution Architect dispatch.
>
> 5. **Authorization-citation mismatch**: does this dispatch CITE an ADR, BR, Solution doc, or prior dispatch as authorization for its design? If yes, the Orchestrator MUST read the cited section verbatim and confirm the dispatch design matches. Citation-mismatch (the cited document explicitly says X, the dispatch ships not-X or beyond-X) is a stop-and-question signal regardless of how clean the rest of the dispatch looks. Do not dispatch until the contradiction is resolved by either an upstream amendment (with proper stage gate) or a redesign of the dispatch to match the cited authorization.
>
> The Stage 7 gate CANNOT mark PASS if any of these signals fired during dispatch authoring and was not resolved upstream first. The Orchestrator records the checkpoint evaluation (per dispatch) in the audit-log gate verdict, even when all answers are "No" — the negative confirmation is itself the evidence that the check was run.
>
> **Scope clarifications**:
>
> - This rule does NOT slow down genuine tactical fixes. A 1-line value-default change, a typo correction, a comment update, a test added for an existing scenario — none of these trigger any of the 5 questions. The checkpoint is a 60-second pass for clean dispatches; only architectural drift triggers the full Stage 1-6 redirect.
> - This rule does NOT require pre-dispatch consensus on every chart tweak or code change. Solo Developers can self-evaluate the 5 questions and proceed for clean cases; the Orchestrator only formally invokes the gate when authoring multi-agent dispatches.
> - This rule does NOT retroactively undo merged work. When detection happens after Stage 7 merge (because the checkpoint was added mid-project, or was missed by an Orchestrator under deadline pressure), remediation = retroactive Stage 1-6 formalization, NOT code rollback. The "working software is good" guidance stays.
> - Each "Yes" answer that requires upstream coverage SHOULD cite the specific upstream artifact (BR-N, Solution §N, ADR #N) that authorizes the design. "Yes — covered by Solution §3.2 + BR-145" is acceptable; "Yes — Solution covers it" is not specific enough and shifts gate burden to readers.

## Sibling Rule for Protocol 007

Add the following rule to Protocol 007 (Multi-Agent Pipeline Execution) § Orchestrator Sequencing:

> ### Rule 6 (new) — "Tactical fix" framing is not a pipeline shortcut
>
> When a Stage 7 dispatch's acceptance criteria expand the project's architectural surface area (any "Yes" to questions 1-4 of the Stage 7 Pre-Flight Architectural-Pivot Checkpoint in Protocol 001), the dispatch ceases to be tactical and MUST follow the full Stage 1-6 pipeline before Stage 7 ships — regardless of how the dispatch was initially titled, scoped, or framed.
>
> The Orchestrator records the scope-expansion detection in the audit-log gate verdict. Subsequent retroactive Stage 1-6 dispatches (if the scope expansion is detected after Stage 7 merge rather than at pre-flight) are dispatched in standard sequence (Analyst → Specifier → Test Architect → Solution Architect → Developer revalidation) and tracked as a remediation chain in the audit-log.

## Cross-references to sibling prompts

This prompt is the fourth in a coherent methodology-hardening family addressing the same structural failure mode — implicit / undocumented / unverified architecture surviving past the gate that should have caught it — at four different gates:

- **Prompt 019** (Architectural Decisions MUST Be Documented; Agents Question, Never Invent): every stage. Prevents agents from inferring rationale post-hoc and propagating drift.
- **Prompt 020** (Stage 6 gate: solution docs MUST enumerate every external service consumed): Stage 6. Catches cross-namespace / cross-Helm-release addressing mismatches at design time.
- **Prompt 021** (Stage 5 gate: test specs for runtime BRs MUST include end-to-end scenarios): Stage 5. Catches runtime behavior that was scoped out of unit tests and never end-to-end exercised.
- **Prompt 022** (Stage 7 pre-flight: architectural-pivot checkpoint): Stage 7 pre-flight. Catches Stage 7 dispatches whose acceptance criteria expand architectural surface area beyond their stated tactical scope.

Each prompt addresses a distinct gate. Together they form a defense-in-depth pattern: the same architectural drift that escapes one gate may still be caught by another. Prompt 022 is the latest-in-the-pipeline check and the last opportunity before merged code locks in the drift.

## Expected Output

1. **Protocol 001** — new "Stage 7 Pre-Flight Architectural-Pivot Checkpoint" subsection added to Stage 7 gate criteria, placed after the existing Stage 7 deliverable and gate-criterion definitions. Include the 5-question checklist verbatim.
2. **Protocol 007** — new "Rule 6 — Tactical fix framing is not a pipeline shortcut" added to § Orchestrator Sequencing. Include the scope-expansion-detection record requirement.
3. **README.md** — Protocol 001 + Protocol 007 entries updated to reflect the new gate criterion and sequencing rule.
4. **Optional**: a template `templates/stage7-dispatch-with-preflight.md` showing the canonical dispatch-artifact pre-flight section (5 questions answered explicitly + citations for each "Yes").
5. **Optional**: an Orchestrator playbook entry showing the 60-second pre-flight checklist execution as a dispatch-authoring habit (questions → halt-conditions → audit-log recording).

## Read Before Editing

- `protocols/001-documentation-driven-development.md` — Stage 7 gate criteria; the natural insertion point for the new pre-flight checkpoint.
- `protocols/007-multi-agent-pipeline-execution.md` — Orchestrator Sequencing rules; the natural insertion point for Rule 6.
- `protocols/003-solution-architecture.md` — § 7 (Cross-Component Service References, added by Prompt 020) cross-references this checkpoint, since RBAC + cross-namespace access checks share the same trigger surface (question 4 of the checkpoint).
- `prompts/019-architectural-documentation.md`, `prompts/020-stage6-gate-cross-component-services.md`, `prompts/021-stage5-gate-runtime-e2e-tests.md` — sibling prompts; preserve the family voice and cross-link from this prompt's published version.

## Triggering Incident Summary (for evidence)

- **Date**: 2026-06-04, post-Phase-8 v1 single-user-dev-mode remediation work.
- **Project**: Mila (Kubernetes-native agentic control plane; the project of origin for Prompts 019-021).
- **Sequence** (3-dispatch escalation; full forensic trail in Mila's `docs/coordination/audit-log.md` + workflow `wf_cfa63306-47d` synthesis):
  - A **clean tactical dispatch** (D-117 in Mila parlance — Runtime Developer, 1-line chart flip) properly authorized by an upstream Specifier amendment (D-116 → BR-MILA-173 asymmetric `readOnly`). Stage 4 → Stage 7 traceability intact. Minor flag: skipped Stage 5 test-spec amendment.
  - The **next dispatch** (D-118 — Runtime, Stage 7) was framed as "fix entry-pod mount gap" — but its acceptance criteria silently introduced (1) a new Kubernetes Secret resource kind (`mila-claude-auth`), (2) a substrate change (hostPath → Secret + emptyDir), (3) removal of `values.yaml` fields without BR amendment, AND (4) cited an ADR (#042) whose verbatim text explicitly mandated hostPath retention. All five checkpoint signals fired. The Orchestrator dispatched anyway.
  - The **next dispatch** (D-119 — Dashboard, Stage 7) was framed as "claude-auth UI" — but introduced a new HTTP API surface (`GET/PUT/DELETE /api/v1/claude-auth`), a new RBAC binding (dashboard SA writes Secrets in `mila-system`), and a new architectural pattern (dashboard pod becomes a Secret writer) directly contradicting a BR (BR-MILA-187) that said the broker is the sole Secret writer. Zero upstream coverage at Stages 1, 3, 4, 5, 6.
  - **Result**: working software, but the Mila documentation pipeline now described a credential architecture that didn't match the deployed chart. Forensic audit cost: 1 retroactive Analyst dispatch (D-120, originally proposed) + 3 amendment dispatches (Specifier, Test Architect, Solution Architect) + 1 ADR amendment + 1 elevation amendment (after manual test surfaced a runtime gap missed by D-120 itself) + a second remediation cascade (D-124-D-129) once empirical end-to-end validation revealed the documentation was still incomplete. Total: 9+ retroactive dispatches to formalize what the original 3 Stage 7 dispatches should have triggered upstream first.
- **Cost without this prompt**: each future project running Protocol 001 + Protocol 007 will eventually hit the same pattern. The trigger is universal (any tactical Stage 7 dispatch under deadline pressure, with an Orchestrator who doesn't cross-check cited authorizations). The fix without this prompt is the same as Mila's fix — retroactive multi-dispatch remediation cascade per incident.
- **Cost with this prompt**: 60 seconds per dispatch (5 questions; "No" is the common case), with the Orchestrator halting and redirecting upstream when any answer is "Yes" without upstream coverage. The Mila case (had this prompt been in force at D-118 authoring time) would have halted D-118 immediately on questions 1+2+5 and required ADR amendment + BR amendment + Solution doc + Contract Registry entry before any Stage 7 work. D-119's RBAC + new contract surface (questions 3+4) would have similarly halted. No cascade; no retroactive work; documentation and code stay in sync.

## Mila evidence pack (for the integration reviewer)

The Mila project's `docs/coordination/audit-log.md` 2026-06-04 + 2026-06-05 entries contain the full forensic chain. Specifically:

- `D-117` gate verdict (minor — Stage 5 skipped; one signal — the test-spec staleness — visible but not blocking).
- `D-118` Developer + Orchestrator entries (major — substrate change + new resource kind + values.yaml removal + citation-mismatch; all five signals fired).
- `D-119` dispatch entry + downstream-flagged contradictions with BR-MILA-187 (critical — new HTTP contract + new RBAC binding + new architectural pattern contradicting existing BR).
- `D-120` Analyst retroactive formalization (post-audit remediation pass 1 — Analyst was unaware of the runtime gap; formalized what shipped, not what was needed to work).
- `D-124` Analyst retroactive formalization (post-audit remediation pass 2 — after empirical end-to-end manual test revealed the documentation was still incomplete; expanded ADR + UJ + SOW for the actually-required substrate).
- `D-125`-`D-129` cascade (Specifier + Runtime + Dashboard + Test Architect + Integration — propagating the corrected ADR through the BR layer + code + tests + binding acceptance gate).

Companion memory items in Mila's session memory (file-based memory at `~/.claude/projects/-home-themaster-workspace-agentic/memory/`):

- `feedback_name_the_class_not_the_instance.md` — names this anti-pattern (tactical-fix framing concealing architectural choices).
- `feedback_document_arch_decisions.md` — questioning vs inventing (Prompt 019 sibling).
- `feedback_silent_3way_merge_data_loss.md` — adjacent failure class (parallel dispatches in shared working tree; orthogonal to this prompt but discovered in the same incident chain).

## Status

Drafted 2026-06-04. Submitted to evocatio software-forge prompts/ 2026-06-04. Awaiting maintainer review + integration into Protocol 001 + Protocol 007. Mila project will adopt the checkpoint immediately in subsequent dispatches regardless of upstream integration timing (see Mila CLAUDE.md update + Orchestrator playbook update post-2026-06-04).
