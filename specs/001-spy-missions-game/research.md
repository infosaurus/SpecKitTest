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
