# research.md

## Decisions (resolved NEEDS CLARIFICATION)

### Resource Tick Cadence
- Decision: Configurable `resourceTick` using ISO 8601 duration strings. Production default: `PT24H` (24-hour tick). For testing and short game modes, planners may set a shorter duration (e.g., `PT5M`).
- Rationale: Configurability enables fast QA and different game modes while keeping a sensible production default. ISO 8601 durations are machine-parsable.
- Alternatives considered: fixed calendar-day or fixed 24-hour aligned to local midnight. Rejected due to timezone fairness and testing friction.

### Game End Condition
- Decision: Hybrid rule: game ends when either a) a player reaches `targetVictoryPoints` OR b) `maxDays` elapsed since `startAt`. Both parameters are configurable per-game; default `maxDays=7`, `targetVictoryPoints=50`.
- Rationale: Hybrid supports both predictable-duration games and early competitive finishes.
- Alternatives considered: first-to-X only (may produce too-short games) or only fixed-duration (may drag); hybrid provides flexibility.

### Investigation Intel Model
- Decision: Tiered reveal model.
  - Low-investment investigations return coarse/probabilistic intel (type hints, confidence score).
  - Higher-investment investigations (costlier agents or aggregated probes) can produce deterministic/full reveal.
- Rationale: Tiered model balances uncertainty and strategic choices; easier to implement than fully probabilistic models that require complex RNG and balancing.
- Alternatives considered: pure deterministic (too blunt) or purely probabilistic (adds RNG complexity and testing friction).

## Implementation Notes
-- `resourceTick` should be stored in game config as an ISO 8601 duration string and translated into a `nextTickAt` UTC timestamp by server logic.
-- Read models may lag behind writes (eventually consistent). Where immediate consistency is required (e.g., joining a game), provide synchronous read-after-write via the write-side snapshot or use distributed transactions.

## Persistence Decision: File-based storage (MVP)

- Decision: Use a simple file-based append-only event store for the write model and JSON files for read-model projections. Store events per-game under `./data/events/{matchId}.events` (newline-delimited JSON) and read-model snapshots under `./data/read-models/games/{matchId}.json`.
- Rationale: The project requirement is to avoid RDBMS systems. A file-based append-only store is lightweight, easy to reason about, simple to test locally and in CI, and supports eventual migration to a dedicated event store or RDBMS later.
- Implementation notes:
  - Events are written with an incremental sequence number and a timestamp. Writers should use file locks or atomic append operations to ensure consistency.
  - Idempotency can be implemented by storing an `idempotency` file per-commander or per-request-key that maps keys to event sequence numbers or match IDs.
  - Snapshots: to speed reads and bootstrapping, periodically write a snapshot JSON of the aggregate state (e.g., every N events) to `./data/snapshots/{matchId}.snapshot.json` and rebuild state by applying events after the latest snapshot.
  - Read projections: projection handlers consume appended events and update JSON read-model files under `./data/read-models/`.
  - Concurrency: use optimistic sequencing and simple advisory file locks for writes; reject conflicting writes with a domain error and surface `409 Conflict` at the API layer when necessary.

- Alternatives considered:
  - SQLite: single-file DB offering SQL semantics; rejected to respect the "no RDBMS" constraint and to keep the persistence model explicit (append-only events).
  - PostgreSQL / RDBMS: rejected per user constraints and for MVP simplicity.

## Operational notes for file-based DB

- Backups: copy the `./data/` directory regularly and include it in CI test snapshots for reproducible tests.
- Scaling: file-based store is suitable for local development, tests, and low-throughput MVP. For larger scale, migrate to an event store or database.
- Migration: the append-only JSON events format is intentionally straightforward to enable migration into tables or an event store if required later.
