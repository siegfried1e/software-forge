# Prompt 016 — Add Retroactive Formalization Rule

## Context

During integration testing, small defects often surface that get fixed ad-hoc: a `kubectl patch`, a manual file edit, a config tweak during deployment. These fixes bypass the documentation-driven pipeline — no BR, no test specification, no dispatch, no gate. They work, but they create invisible technical debt: undocumented behavior that no test covers and no specification describes.

**Evidence from a multi-phase integration:**

| Ad-hoc fix | What happened | Consequence |
|------------|--------------|-------------|
| `securityContext.runAsUser: 1000` | Integrator patched the Helm deployment manually in-cluster | Not in Helm source. Next `helm upgrade` may regress it. |
| Volume mount for `/etc/app/config` | Integrator fixed the volumeMount config in-cluster | Fix partially in Helm values but volume path was corrected ad-hoc |
| Custom resource definition missing from operator Helm chart | Orchestrator copied the file directly | Correct fix, but no dispatch, no BR documenting "operator Helm must include all CRDs" |
| `kafka` in transport provider dropdown | Orchestrator edited TransportStep.tsx directly | No BR, no test spec — if someone regenerates the component from specs, the option disappears |

In every case, the fix was correct and necessary. In every case, the pipeline was bypassed. The pattern emerged: "we'll formalize it later." **This prompt ensures 'later' is defined and enforced.**

## Prompt

Add a **Retroactive Formalization** rule to Protocol 006 (Phased Multi-Component Delivery), since it governs the lifecycle of changes across phases. The rule should:

1. **Define the problem**: during integration testing (Stage 7) or post-gate deployment, ad-hoc fixes are sometimes applied that bypass the documentation-driven pipeline (Protocol 001 Stages 1-6). These fixes are not inherently wrong — they're often necessary to unblock a gate. But they create undocumented behavior.

2. **Define the obligation**: when a change bypasses the pipeline, it incurs a **formalization debt**. The Orchestrator MUST track each such change and ensure it is retroactively formalized before the next phase's Stage 7 begins. Formalization means: a BR exists that describes the behavior, a test specification covers it, and a test in the codebase verifies it.

3. **Define two resolution paths**:
   - **Backfill dispatch** (immediate): the Orchestrator dispatches the Specifier to write a BR + the Test Architect to write a test spec + the Developer to write the test. Used when the change is isolated and the debt should be cleared now.
   - **Bundle into next phase** (deferred): the ad-hoc fixes are listed as "Phase N stabilization" items in the next phase's Analyst SOW (Stage 1). The Specifier, Test Architect, and Developer cover them alongside the new phase's work. Used when multiple small fixes accumulate and individual backfill dispatches would be overhead.

4. **Define what the Orchestrator tracks**: the decision log or a dedicated "formalization debt" section in the coordination records should list each ad-hoc fix with: what was changed, who changed it, when, and whether the formalization is pending or complete.

5. **Define the deadline**: formalization debt from Phase N MUST be resolved before Phase N+1 Stage 7 begins. It MAY be resolved at any earlier point. It MUST NOT be carried across two phase boundaries (i.e., debt from Phase N cannot still be open when Phase N+2 starts).

6. **Note on prevention**: the best formalization is the one you don't need. The Orchestrator should prefer dispatching agents for fixes (even small ones) over applying fixes directly. But when ad-hoc fixes do happen — especially under time pressure during integration — the retroactive path exists as a safety net, not an excuse.

## Expected Output

An update to Protocol 006 — new rule or subsection (e.g., "Rule 9: Retroactive Formalization" or a subsection under the existing phase gate rules). Concise (under 40 lines). Cross-reference Protocol 001 (the pipeline that was bypassed) and Protocol 007 (the Orchestrator's tracking responsibility).

Optionally, also add a note to Protocol 007's Orchestrator Responsibilities section referencing the retroactive formalization obligation.

## Read Before Editing

- `protocols/006-phased-multi-component-delivery.md` — phase gate rules, Master SOW lifecycle
- `protocols/007-multi-agent-pipeline-execution.md` — Orchestrator responsibilities
- `protocols/001-documentation-driven-development.md` — the 7-stage pipeline that ad-hoc fixes bypass

## Scope Notes

- Do NOT weaken existing rules. The retroactive path is a recovery mechanism, not an alternative to the pipeline.
- The rule should make clear that bypassing the pipeline is tolerated (because reality is messy) but never normalized (because traceability is the pipeline's core value).
