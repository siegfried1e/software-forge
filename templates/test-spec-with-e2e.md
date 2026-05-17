# Test Specification with E2E Scenario — Template

Use this template for BRs that specify runtime behavior (Protocol 002, Rule 8). Copy to `docs/test-specifications/BR-PROJ-NNN_<ShortName>_test-spec.md`.

---

# Test Specification — BR-PROJ-NNN: <Title>

## BR Reference

BR-PROJ-NNN

## Coverage Summary

- Success scenarios: <count>
- Failure scenarios: <count>
- Edge case scenarios: <count>
- E2E scenarios: <count>
- Out of scope: <list from BR's "Does not cover">

## Scenarios

### S-001: <Unit-level Scenario Name>

**Category**: Success | Failure | Edge
**Input**: <concrete input values>
**Expected Outcome**: <expected result for success, exact error message for failure>
**BR Reference**: BR-PROJ-NNN
**Notes**: <optional — context for the Developer at Stage 7>

### S-NNN: <E2E Scenario Name> `[E2E]`

**Category**: E2E (Integration)
**Runtime Entry Point**: <e.g., "operator reconcile loop invoked with new CR", "worker claims a queue message", "BFF receives HTTP POST /api/x">
**Runtime Exit Point**: <e.g., "CR reaches terminal phase", "completion message published on subject X", "HTTP 200 returned with body Y">
**Cross-Component Path**: <e.g., "operator → CR → worker → provider → external API → response → completion publish">
**Observable Signal**: <e.g., "CR.status.phase == 'Completed'", "message published on subject `app.completion.v1` with payload schema {...}", "HTTP 200 with JSON body matching schema Z">
**Input**: <concrete trigger — what the test arranges to cause the entry-point event>
**Expected Outcome**: <observable end state at the exit point>
**BR Reference**: BR-PROJ-NNN
**Notes**: <implementation hints for Stage 7 — e.g., "may be implemented as integration test against a real cluster, or as multi-component test with mocked external API; both satisfy the scenario as long as the runtime path is exercised">

---

## Notes on the E2E Scenario Shape

The five mandatory elements (entry, exit, path, signal, no delegation) come from Protocol 002, Rule 8. The Test Architect specifies *what to observe*; the Developer at Stage 7 chooses *how to execute* (integration test framework, real cluster, mocked boundaries with real runtime under test, etc.).

When a BR's runtime path crosses components owned by different agents, the test spec MUST still include the E2E scenario — the Test Architect does not delegate the runtime path to "another component's scope."
