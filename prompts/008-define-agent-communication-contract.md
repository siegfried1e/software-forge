# Prompt 008 — Define Agent Communication Contract

## Context

Protocol 007 (Multi-Agent Pipeline Execution) defines agent roles, context packages, and gate conditions, but does not define the concrete communication contract between the Orchestrator and agents. It specifies WHAT roles exist and WHAT they produce, but not the artifact format for dispatch, completion, and gate feedback.

This gap was discovered during real pipeline execution: the Orchestrator had no standard way to send mandates to agents or receive structured completion signals back. The transport mechanism varied (human relay, programmatic dispatch, file-based handoff) but the contract for what is exchanged should be constant.

## Prompt

Propose an addition to Protocol 007 (new section) or a new Protocol 008 that defines the communication contract between the Orchestrator and agents.

Before writing, read:
- `protocols/007-multi-agent-pipeline-execution.md` — the current protocol (especially Context Handoff, Gate Enforcement, and Coordination Records sections)
- `protocols/001-documentation-driven-development.md` — the pipeline stages and gates

The proposal must:

1. **Define three communication artifacts** at the contract level, not the transport level:
   - **Dispatch artifact** (Orchestrator → Agent): the mandate. Contains the context package, assigned stages, acceptance criteria, and constraints. This is what the agent reads to know what to do.
   - **Completion artifact** (Agent → Orchestrator): the delivery signal. Contains what was produced (file paths), what status (complete/partial/blocked), what the agent couldn't resolve, and any discovered requirements.
   - **Gate feedback artifact** (Orchestrator → Agent): the evaluation result. Contains pass/fail, deficiency list for rework if failed, and instructions for the next iteration.

2. **Define minimum required fields** for each artifact — not a rigid template, just the contract that any implementation must satisfy.

3. **Be transport-agnostic**. The same contract must work whether:
   - A human copies a prompt to an agent in a separate session
   - An API call spawns an agent subprocess
   - An agent reads a file from a shared directory
   - A message queue delivers the artifact
   - Any future mechanism

4. **Define where artifacts live** as a convention, not a requirement. Suggest a default location in the project structure but allow implementations to override.

5. **Address failure modes**: agent doesn't respond, completion artifact is incomplete, gate feedback is lost, agent produces output without reading the dispatch artifact.

6. **Integrate with existing Protocol 007 concepts**: context packages become the content of the dispatch artifact, gate verdicts become the content of the gate feedback artifact, coordination records log all exchanges.

Rules:
- The protocol must not mention specific tools (Claude Code, ChatGPT, subprocess, etc.)
- The protocol must not assume the Orchestrator and agents run in the same process, machine, or session
- The protocol must not assume synchronous communication
- Keep it minimal — define the contract, not the implementation

## Expected Output

A protocol proposal as a markdown file following the software-forge protocol structure (Status, Purpose, Rules, Gate). Place it at `protocols/008-agent-communication-contract.md`.
