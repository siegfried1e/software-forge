# Prompt 020 — Stage 6 gate: solution docs MUST enumerate every external service consumed by designed components

> **Source**: drafted by Mila Solution Architect (`/home/themaster/workspace/agentic`) on 2026-05-16 after D-095 (Phase 6 stab3 solution doc).
> **To submit**: copy this file to `/home/themaster/workspace/evocatio/software-forge/prompts/020-stage6-gate-cross-component-services.md`, review/refine, and integrate into Protocol 003 (Solution Architecture).

## Context

Phase 6 of the Mila project closed with two friction discrepancies (FD-010, FD-011) that a Stage 6 gate check would not have caught under the current protocols:

- **FD-010**: `mila-nats-creds` Secret — BR-MILA-144 required the Secret in the `mila` namespace, but the `mila-agent-worker` chart templated it in `.Release.Namespace` (= `mila-system`). The BR and the chart were each internally consistent; the conflict was only visible when you compared the cross-component contract to the chart's implementation.
- **FD-011**: `mila-entry` URL — the dashboard BFF hardcoded `mila-entry.mila.svc.cluster.local:8080`, but the `mila-entry` Service is deployed in `mila-system`. The mismatch was silent until runtime.

Both frictions share a structural cause: Phase 6 solution docs described component internals thoroughly (consumers, NATS subjects, Deployment templates, env vars, BR coverage) but did not enumerate the external services each component *consumes across release or namespace boundaries*, nor did they specify what Helm value exposes the URL override and what default the value must carry. The gap was not an agent failure; it was the absence of a gate criterion that would have required the information to be present.

The project-side fix was documented in the D-095 solution doc (`docs/phase-6/solution/stabilization-3.md`): a new BRs (181/182/183) formalize the cross-namespace addressing contract, a lint check enforces it, and a Stage 6 gate amendment spec requires future solution docs to include a "Cross-Component Service References" section. The project-side spec is complete.

The methodology-side fix proposed here would make the same requirement cross-project: any software-forge project running the Protocol 003 pipeline would be required to include this section in its Stage 6 deliverables.

## Prompt

Add a "Cross-Component Service References" requirement to Protocol 003 (Solution Architecture). This requirement ensures that cross-component Service addressing is made explicit at design time rather than discovered during integration testing.

### Part A — Protocol 003 (Solution Architecture): add "Cross-Component Service References" subsection

Protocol 003 defines the required structure of a Stage 6 solution document. Add a new mandatory subsection, positioned after the component design sections and before the BR Coverage table:

> **Cross-Component Service References** (mandatory subsection)
>
> Every solution document whose scope includes component design MUST contain a "Cross-Component Service References" section. The section enumerates every external Service (or other named resource — Secret, ConfigMap, PVC) that the designed component *consumes across Helm release or namespace boundaries*.
>
> For each entry, the section MUST specify:
>
> (a) **Consumed resource**: the name and kind of the external Service or resource.
> (b) **Canonical placement**: the namespace where the provider is expected to run, per the project's ADR or architecture document (e.g., Mila's decision-log #017 for the `mila-system` / `mila` split).
> (c) **Helm override value**: the name of the Helm value in the *consuming* chart that exposes the URL (or namespace component) as configurable, so that non-default placements can be accommodated at install time.
> (d) **Defaults aligned**: confirmation that the chart value default matches the canonical FQDN, and that any source-code fallback (hardcoded env-var default in Rust/Go/TypeScript) matches the chart value default. If the BR mandates fail-to-start-if-unset, state "N/A — env var required; no source fallback."
>
> A Stage 6 gate cannot PASS unless this section is present and contains entries for every cross-release/cross-namespace Service reference introduced or modified by the solution. If the component being designed consumes no external Services across release or namespace boundaries, the section MUST be present and explicitly state "None — this component consumes no cross-namespace Services."
>
> **Why this section prevents real bugs**: when a solution doc omits cross-component Service addresses, downstream agents (Test Architect, Developer) must infer the URL, the namespace, and the override mechanism from observed code or BRs. Inference is not knowledge — it produces subtle mismatches that only appear at runtime (see FD-010, FD-011, Mila Phase 6 stab3 for a concrete example). Requiring the section at Stage 6 surfaces the constraint at design time, when it is cheapest to fix.

### Part B — Protocol 003: update the mandatory section list

The protocol's enumeration of mandatory solution document sections currently reads (approximately): overview, component design, build-deploy, BR coverage, traceability. Add "Cross-Component Service References" to this list, positioned between the component design sections and the BR coverage table.

### Part C — Protocol 001 (Documentation-Driven Development): update Stage 6 gate criteria

Protocol 001 Stage 6 gate criteria should reference the new requirement. Add a bullet:

> - Solution document includes a "Cross-Component Service References" section per Protocol 003, with entries for every cross-release/cross-namespace Service the designed component consumes.

## Expected Output

1. **Protocol 003** — new "Cross-Component Service References" mandatory subsection (~25-40 lines); cross-references Protocol 001 Stage 6 gate and Protocol 007 (agent communication).
2. **Protocol 003** — mandatory section list updated to include the new subsection.
3. **Protocol 001** — Stage 6 gate criteria list updated with one additional bullet.
4. **Optional: a template row** for the cross-component references table, usable in agent prompt templates.

## Read Before Editing

- `protocols/003-solution-architecture.md` — the natural insertion point; check existing mandatory section structure.
- `protocols/001-documentation-driven-development.md` — Stage 6 gate criteria; add one bullet.
- Prompts 016 (retroactive formalization), 019 (architectural documentation) — related context; the requirement proposed here is a specific application of the "make the implicit explicit" principle from both.

## Acceptance

The integration is complete when:
1. `protocols/003-solution-architecture.md` contains a "Cross-Component Service References" subsection under solution document structure, with the four mandatory fields listed above.
2. `protocols/001-documentation-driven-development.md` Stage 6 gate has a bullet referencing this subsection.
3. The change is internally consistent: a Stage 6 deliverable following the updated Protocol 003 template would produce a section that satisfies the updated Protocol 001 gate.

## Triggering Incidents

- **FD-010** and **FD-011**: Mila Phase 6 cross-namespace addressing discrepancies (2026-05-16, during D-091/D-093 closure).
- **ADR #017**: Mila decision-log — `mila-system` (control plane) / `mila` (data plane) namespace split; the authoritative placement table that chart defaults should reflect.
- **D-095 solution doc**: `docs/phase-6/solution/stabilization-3.md` — the project-side stab3 fix; includes the "Cross-Component Service References" section as a dogfood instance of the proposed criterion.
