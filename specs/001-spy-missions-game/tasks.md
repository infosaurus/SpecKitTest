---
description: "Task list for implementing 'Start a new game' (Commander)"
---

# Tasks: Start a new game (Commander)

**Input**: spec.md, plan.md, data-model.md, contracts/openapi.yaml, research.md

## Phase 1: Setup (Shared Infrastructure)

- [ ] T001 [P] Create project skeleton directories: src/domain, src/backend, frontend, projections, tests (create directories at repository root)
- [ ] T002 [P] Initialize .NET solution and projects: create solution file and projects for F# domain (src/domain), ASP.NET Core API (src/backend) and placeholder frontend (frontend/package.json)
- [ ] T003 [P] Add initial CI workflow: .github/workflows/ci.yml with build/test steps for domain and backend
- [ ] T004 [P] Add quickstart and developer notes at specs/001-spy-missions-game/quickstart.md

## Phase 2: Foundational (Blocking Prerequisites)

- [ ] T005 Implement authentication scaffolding (JWT + commander role) in src/backend (src/backend/Infrastructure/Auth/*)
- [ ] T006 [P] Setup PostgreSQL connection and EF Core migrations in src/backend/Infrastructure (configure connection string and migrations folder)
- [ ] T007 [P] Add event-store abstraction (interface) in src/backend/Infrastructure/EventStore (placeholder implementation using Postgres append-only table)
- [ ] T008 [P] Add domain test harness and project: tests/domain (Expecto) and test project files under src/domain/tests

## Phase 3: User Story US1 - Start a new game (Priority: P1) 🎯 MVP

**Goal**: Commander can create a game via API; domain aggregate enforces invariants and emits events; initial game state persisted and read projection available.

**Independent Test**: Contract test for POST /api/v1/games and an integration test that creates a game and verifies read-model state.

### Tests (Test-First)

- [ ] T009 [P] [US1] Add contract tests for POST /api/v1/games in tests/contract/test_games_create.cs referencing specs/001-spy-missions-game/contracts/openapi.yaml
- [ ] T010 [P] [US1] Add unit tests (Expecto) for GameSession Decide/Evolve behavior in tests/unit/domain/test_gamesession.fs

### Domain (F#) Implementation

- [ ] T011 [P] [US1] Create command and DTO definitions in src/domain/commands/StartGameCmd.fs
- [ ] T012 [P] [US1] Implement GameSession aggregate types and state in src/domain/GameSession.fs
- [ ] T013 [US1] Implement Decide function (State -> Command -> Result<Event list, DomainError>) in src/domain/GameSession.fs
- [ ] T014 [US1] Implement Evolve function (State -> Event -> State) and event types in src/domain/GameSession.fs
- [ ] T015 [US1] Add repository port interface IGameSessionRepository in src/domain/Ports/IGameSessionRepository.fs

### Infrastructure & API

- [ ] T016 [US1] Implement ASP.NET Core controller POST /api/v1/games in src/backend/Controllers/GamesController.cs that validates auth, maps request to StartGameCmd, calls application layer, and returns 201 with Location header
- [ ] T017 [US1] Implement event persistence adapter in src/backend/Infrastructure/EventStore/PostgresEventStore.cs (persist events and snapshots)
- [ ] T018 [US1] Wire domain port to infrastructure adapter in src/backend/Startup (dependency injection)
- [ ] T019 [US1] Implement projection handler to populate read tables and create GET /api/v1/games/{gameId} in src/backend/Controllers/GamesQueryController.cs and src/backend/Projections/GameProjection.cs

### Integration

- [ ] T020 [US1] Add integration test (Testcontainers) in tests/integration/test_games_integration.cs verifying end-to-end game creation and projection population

## Phase 4: Polish & Cross-Cutting Concerns

- [ ] T021 [P] Add logging & observability: structured logs for command handling, metrics for game creations in src/backend/Observability
- [ ] T022 [P] Add API contract validation to CI: fail if OpenAPI no longer matches controllers (compare contracts/openapi.yaml)
- [ ] T023 [P] Update documentation and examples in specs/001-spy-missions-game/ (spec.md, plan.md, quickstart.md)
- [ ] T024 [P] Security review: ensure commander role enforcement and rate-limiting on POST /api/v1/games

## Dependencies & Execution Order

- Phase 1 tasks (T001-T004) can start immediately and run in parallel.
- Phase 2 (T005-T008) MUST complete before domain work begins; they can run in parallel where marked [P].
- Phase 3 (US1) tasks depend on Phase 2 completion; within US1, Tests (T009-T010) should be written before implementation tasks T011-T015.
- Integration tasks (T020) run after infrastructure and API wiring (T016-T019) complete.

## Parallel Execution Examples

- While T006 (DB setup) runs, a separate engineer can implement T011-T014 (domain types and Decide/Evolve) and T010 (unit tests) in parallel, using an in-memory test double for persistence.
- T016 (controller) and T017 (event store) can be implemented in parallel if clear interface (IGameSessionRepository) is defined first (T015).

## Implementation Strategy (MVP-first)

- Focus on a minimal end-to-end: implement F# GameSession aggregate with StartGame command, an in-memory event store adapter, and a minimal ASP.NET controller to exercise the flow. Cover with unit, contract, and a single integration test.
- After MVP passes, replace in-memory adapter with PostgresEventStore and add projections for read models.

## Files to be created/updated (high-value references)

- specs/001-spy-missions-game/spec.md
- specs/001-spy-missions-game/plan.md
- specs/001-spy-missions-game/contracts/openapi.yaml
- specs/001-spy-missions-game/quickstart.md
- src/domain/** (F# library)
- src/backend/** (ASP.NET Core API)
- tests/contract/**, tests/unit/**, tests/integration/**
