# data-model.md

## Entities and Aggregates

### GameSession (Aggregate Root)
- Fields: `matchId: Guid`, `players: Player list`, `status: MatchStatus`, `config: MatchConfig`, `resourcePools: Map<PlayerId, ResourcePool>`, `events: DomainEvent list`, `createdAt: DateTimeUtc`, `nextTickAt: DateTimeUtc`

### Player
- Fields: `playerId: string`, `faction: Faction`, `state: PlayerState`, `victoryPoints: int`

### Mission
- Fields: `missionId: Guid`, `owner: PlayerId`, `cost: ResourceBundle`, `accumulated: ResourceBundle`, `state: MissionState`, `visibility: VisibilityRule`, `reward: int`

### Agent
- Fields: `agentId: Guid`, `owner: PlayerId`, `location: AgentLocation`, `abilities: Ability list`, `cooldownUntil: DateTimeUtc option`

### ResourcePool / ResourceBundle
- Resource types: `Credits`, `Force`, `Stealth`, `Investigation`, `Tech` — represented as a record or map from ResourceType -> int.

## Commands (write model inputs)
- `StartMatch(command: StartMatchCmd)` — create GameSession with config
- `InvitePlayer(cmd: InvitePlayerCmd)`
- `JoinMatch(cmd: JoinMatchCmd)`

## Events (domain events emitted by Decide)
- `MatchStarted`, `PlayerInvited`, `PlayerJoined`, `MissionDeployed`, `AgentDeployed`, `ResourcesTicked`, `MissionResolved`, `InvestigationResulted`

## Aggregate design (Decide / Evolve)

The aggregate exposes two pure functions:

- `Decide : State -> Command -> Result<Event list, DomainError>`
- `Evolve : State -> Event -> State` (applied iteratively for event lists)

Workflow example:

1. Application receives `StartMatchCmd` and calls `Decide currentState cmd`.
2. `Decide` returns events like `[MatchCreated; PlayerInvited...]`.
3. Application persists events and applies `Evolve` on each event to compute new state snapshot to persist.

## Validation rules
- No negative resources in ResourcePool.
- `maxPlayers` must be >= 2 and <= configured maximum (for MVP set to 2).
- `resourceTick` must be a valid ISO 8601 duration.

## Persistence / Storage (File-based)

- Write model: append-only per-match event files stored at `./data/events/{matchId}.events` where each line is a JSON encoded domain event with a sequence number and timestamp.
- Snapshots: optional aggregate snapshots written to `./data/snapshots/{matchId}.snapshot.json` to speed recovery and reads.
- Read model: projections update JSON files under `./data/read-models/games/{matchId}.json` which the API reads to serve queries.
- Idempotency: implement idempotency-key tracking in `./data/idempotency/{commanderId}.{key}.json` mapping to the resulting `matchId` to support idempotent `POST /api/v1/games` requests.

Notes: This file-based model avoids RDBMS dependencies for MVP, simplifies local development and CI, and maps naturally to the event-sourced aggregate design described above.
