# Implementation Plan: Start a new game (Commander)

**Branch**: `001-spy-missions-game` | **Date**: 2026-01-15 | **Spec**: ../spec.md
**Input**: Feature specification from `/specs/001-spy-missions-game/spec.md`

## Summary

Implement the backend API and domain core to support Commander-driven game creation. The domain layer is implemented in F# as a library following Onion Architecture and DDD tactical patterns. The API is ASP.NET Core (C#) exposing HTTP contracts. Read/write separation (CQRS) will be used: commands handled via the write model (F# aggregates) and queries served from optimized read projections.

## Technical Context

**Language/Version**: Backend: C# (.NET 8) for API; Domain: F# (F# 7 / .NET 8)
**Primary Dependencies**: ASP.NET Core, FSharp.Core, EntityFrameworkCore (Npgsql provider), MediatR (for command dispatch), OpenAPI/Swashbuckle
**Storage**: PostgreSQL for authoritative state and read-model projections
**Testing**: xUnit (C#), Expecto (F#) for unit tests; integration/contract tests using testcontainers + HTTP contract tests
**Target Platform**: Linux containers (Docker) and Windows development; CI/CD via GitHub Actions
**Project Type**: Web app (backend + frontend)
**Performance Goals**: N/A for MVP (document later)
**Constraints**: Domain layer MUST be a pure F# library (no mutable singletons, no OO classes in domain), follow immutability and rich types.

## Constitution Check

- **Library-First**: YES — the domain is a standalone F# library (src/domain) with a public API composed of commands/events and an in-process Decide/Evolve interface.
- **Interface Contracts**: YES — API contracts will be OpenAPI; `POST /api/v1/games` is the canonical contract for this story (see /specs/.../contracts/openapi.yaml).
- **Test-First**: YES — tests will be authored for aggregates (Expecto) and API contract tests (xUnit) before implementation.
- **Integration & Contract Testing**: YES — add contract tests under `tests/contract` and integration tests using Testcontainers for PostgreSQL.
- **Observability & Versioning**: Logs at INFO for commands and WARN/ERROR for failures; expose request/response IDs and metrics; semantic versioning for libraries.
- **Architecture Style**: Onion Architecture — domain library has no infrastructure dependencies; application and infrastructure layers depend inwards.
- **CQRS / Read-Write Separation**: YES — commands handled by F# aggregates and events projected into read tables; document consistency guarantees (eventual consistency for reads by default).
- **Domain-Driven Design (Tactical)**: YES — Aggregates with Decide/Evolve pattern; Aggregates enforce invariants and produce events; domain events drive projections.

## Project Structure

```text
src/
  domain/             # F# library containing aggregates, commands, events, and business rules
  backend/            # ASP.NET Core API (C#) wiring commands to domain and serving queries
  projections/        # projection handlers (can be in backend or separate worker)
frontend/             # React + TypeScript UI
tests/
  unit/
  integration/
  contract/
```

## Phase Plan

Phase 0: Research — produce `research.md` resolving open clarifications (resource tick, end condition, intel model).

Phase 1: Design — generate `data-model.md`, `contracts/` (OpenAPI), `quickstart.md`.

Phase 2: Implementation — scaffold backend project, F# domain library, add tests (Test-First), implement `POST /api/v1/games` and persistence.
# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]
**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: [e.g., Python 3.11, Swift 5.9, Rust 1.75 or NEEDS CLARIFICATION]  
**Primary Dependencies**: [e.g., FastAPI, UIKit, LLVM or NEEDS CLARIFICATION]  
**Storage**: [if applicable, e.g., PostgreSQL, CoreData, files or N/A]  
**Testing**: [e.g., pytest, XCTest, cargo test or NEEDS CLARIFICATION]  
**Target Platform**: [e.g., Linux server, iOS 15+, WASM or NEEDS CLARIFICATION]
**Project Type**: [single/web/mobile - determines source structure]  
**Performance Goals**: [domain-specific, e.g., 1000 req/s, 10k lines/sec, 60 fps or NEEDS CLARIFICATION]  
**Constraints**: [domain-specific, e.g., <200ms p95, <100MB memory, offline-capable or NEEDS CLARIFICATION]  
**Scale/Scope**: [domain-specific, e.g., 10k users, 1M LOC, 50 screens or NEEDS CLARIFICATION]

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

The following checks MUST be documented in this plan and verified before advancing:

- **Library-First**: Confirm the feature is scoped as a self-contained library/module or
  justify why it is not applicable.
- **Interface Contracts**: Document public contracts (CLI/HTTP/gRPC) and examples for
  input/output and error formats.
- **Test-First**: Link to unit tests that will be created; for cross-boundary work,
  include integration/contract test plans.
- **Integration & Contract Testing**: If changing schemas or contracts, include test
  fixtures and validation steps.
- **Observability & Versioning**: Declare required logs/metrics and planned semantic
  versioning for releases or breaking changes.
 - **Architecture Style**: Declare the chosen architecture style (Onion Architecture expected)
   and justify deviations. Document layering and dependency directions.
 - **CQRS / Read-Write Separation**: If applicable, document read model projections,
   command handling, and consistency expectations (eventual vs strong consistency).
 - **Domain-Driven Design (Tactical)**: If the domain uses DDD patterns, document Aggregates,
   Entities, Value Objects, and Domain Events and their boundaries.

Reference: `.specify/memory/constitution.md` for full principle definitions.

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
# [REMOVE IF UNUSED] Option 1: Single project (DEFAULT)
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# [REMOVE IF UNUSED] Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# [REMOVE IF UNUSED] Option 3: Mobile + API (when "iOS/Android" detected)
api/
└── [same as backend above]

ios/ or android/
└── [platform-specific structure: feature modules, UI flows, platform tests]
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
