# Protocol 005 — Cross-Project Integration Contracts

## Status

Active

## Purpose

Define how separate projects expose and consume each other's capabilities. When one project needs to interact with another — whether via message bus, API, shared schema, or tool interface — both sides MUST agree on a contract before implementation begins.

This protocol governs the boundary between projects, not the internals of either one.

## The Problem

Multi-project systems break when:
- Project A changes a message format without telling Project B
- Two projects implement the same integration differently (one expects JSON, the other sends protobuf)
- Integration details are buried in code rather than documented as a shared contract
- A new project wants to integrate but has no specification to work from

## Contract-First Integration

### Principle

Before any cross-project code is written, a **contract document** MUST exist that both sides agree on. The contract is the source of truth — not the implementation.

### What a Contract Covers

Every integration contract MUST define:

1. **Transport** — how messages travel (NATS subjects, HTTP endpoints, gRPC, file system, etc.)
2. **Schema** — the exact structure of messages or requests/responses (JSON schema, protobuf, OpenAPI, etc.)
3. **Subjects / Endpoints** — the addressing convention (topic hierarchy, URL patterns, etc.)
4. **Authentication** — how the caller identifies itself (API key, mTLS, token, none)
5. **Error Handling** — what errors look like and how the caller should respond
6. **Examples** — at least one concrete request/response pair that can be copy-pasted to verify integration

### What a Contract Does NOT Cover

- Internal implementation details of either project
- Technology stack choices within a project
- Deployment topology (unless it affects the contract)

## Contract Document Structure

Contracts live in the consuming project's `docs/contracts/` directory, named after the providing project:

```
docs/contracts/<provider-project>.md
```

Each contract follows this template:

```markdown
# Integration Contract: <Consumer> ↔ <Provider>

## Overview
One sentence describing the integration.

## Transport
Protocol, connection details, addressing.

## Message Schema

### Request / Publish
JSON schema or type definition with field descriptions.

### Response / Reply
JSON schema or type definition with field descriptions.

## Subject / Endpoint Convention
Naming patterns, wildcards, versioning.

## Authentication
How credentials are provided.

## Error Handling
Error format, retry semantics, classification.

## Examples

### Example 1: <Scenario>
Request:
```json
{ ... }
```
Response:
```json
{ ... }
```

## Versioning
How breaking changes are communicated.
```

## Rules

1. **Contract before code.** No cross-project integration code SHOULD be written without a contract document.
2. **Both sides own the contract.** Changes to the contract MUST be communicated to all consuming projects.
3. **Examples are mandatory.** Every contract MUST include at least one copy-pasteable example.
4. **Contracts are versioned.** Breaking changes MUST increment a version or be flagged as breaking.
5. **Test against the contract, not the implementation.** Integration tests SHOULD validate contract compliance, not internal behavior.
6. **Contracts are discoverable.** A project's README or SOW SHOULD reference its contracts.
