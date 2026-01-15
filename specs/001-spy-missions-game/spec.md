# Feature Specification: Spy Missions — 2‑player strategy game

**Feature Branch**: `001-spy-missions-game`  
**Created**: 2026-01-13  
**Status**: Draft  
**Input**: User description: "Build a web-based 2-player strategy game where each player commands a faction. Players undertake missions to gather intelligence, sabotage the opponent's operations, and expand their influence. Players have a limited number of resources that regenerate over time. The objective is to accumulate the most victory points by the end of the game through successful missions and strategic plays."

## Constitution Compliance (mandatory)

All specs MUST include the following explicit statements or links:

- **Library-First applicability**: [Yes/No] and justification if No.
- **Public contract description**: link or inline description of CLI/API contract.
- **Testing approach**: unit tests + any required integration/contract tests.
- **Observability & Versioning**: notes on expected logs/metrics and versioning plan.

Reference: `.specify/memory/constitution.md` for details.

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.
  
  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
  Think of each story as a standalone slice of functionality that can be:
  - Developed independently
  - Tested independently
  - Deployed independently
  - Demonstrated to users independently
-->

### User Story 1 - Play a complete 2‑player match (Priority: P1)

Two players create or join a match, deploy missions and agents, receive daily resources, and play until the match end condition is reached. The system tallies victory points and declares a winner.

**Why this priority**: Core game loop — delivers full gameplay and validates core mechanics.

**Independent Test**: Create a match with two test accounts, simulate resource ticks, perform mission deployments and resolves, and verify victory points and final winner are computed correctly.

**Acceptance Scenarios**:

1. **Given** two players in a match, **When** both deploy/resolve missions and agents over the match duration, **Then** the system records actions, updates resources, assigns victory points, and declares a single winner at end.
2. **Given** one player resolves a mission, **When** resolution completes, **Then** the owner's resources decrease by the mission cost and victory points (if any) are awarded.

---

### User Story 2 - Spy deployment & investigation (Priority: P2)

A player deploys an `Agent` into the Investigation zone targeting opponent missions; the system returns intel (mission presence, type confidence, or partial reveal), enabling counterplay.

**Why this priority**: Differentiates game from standard resource games — core espionage mechanic.

**Independent Test**: Deploy an agent against a known mission, observe intel payload, and verify effects (e.g., reveal, mark, or trigger counter-mechanic).

**Acceptance Scenarios**:

1. **Given** a player has an agent in the Investigation zone targeting an opponent mission, **When** the investigation resolves, **Then** the investigator receives an intel result consistent with the mission's visibility rules.

---

### User Story 3 - Faction asymmetry and resource management (Priority: P3)

Players select a faction (Shadow Syndicate, Iron Dominion, Warriors of the Earth, Triad of Jade) and leverage faction strengths (stealth, force, guerrilla tactics, economy) to pursue victory.

**Why this priority**: Ensures factions feel distinct and introduces strategic depth.

**Independent Test**: Play multiple matches with identical starting conditions but different faction choices; verify faction abilities materially affect options and outcomes.

**Acceptance Scenarios**:

1. **Given** two players with different factions, **When** similar resource expenditures occur, **Then** faction abilities produce different available actions and plausible divergent outcomes.

---

### Edge Cases

- What happens when a player disconnects mid-match? (Expected: match remains active; disconnected player may be replaced by AI or the match continues on a grace timer.)
- How does the system handle simultaneous mission resolves or competing actions that target the same mission? (Expected: deterministic ordering rule, e.g., timestamp or server-validated sequence.)
- How are ties in victory points resolved? (Expected: defined tie-breaker: most resolved missions, then remaining resources.)

## Requirements *(mandatory)*

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right functional requirements.
-->

### Functional Requirements

- **FR-001**: System MUST allow creation and discovery of a 2‑player match and allow two players to join a single match instance.
- **FR-002**: System MUST expose each player's `Player Surface` consisting of five zones: `Plans`, `Player's missions`, `Opponent's missions` (limited visibility), `Investigation zone`, and `Resource Pool`.
- **FR-003**: System MUST support `Agent` and `Mission` entities; agents can be deployed to missions or the Investigation zone; missions accept resources over time and can be `Resolve`d by their owner once funded.
- **FR-004**: System MUST support five resource types (`Credits`, `Force`, `Stealth`, `Investigation`, `Tech`) per player and allow spending these to deploy entities.
- **FR-005**: System MUST regenerate player resources on a periodic cadence (resource tick) so players receive additional resources over time. [NEEDS CLARIFICATION: resource tick frequency — "daily" was specified; confirm exact cadence and whether it should be adjustable for testing or game modes]
- **FR-006**: System MUST compute and persist victory points for mission resolutions and other scoring events and declare a winner when the match end condition is met. [NEEDS CLARIFICATION: match end condition — fixed duration (days), first-to-X points, or player agreement?]
- **FR-007**: System MUST provide an `Investigation` result when agents probe opponent missions; this result may be deterministic or probabilistic depending on mission/faction abilities. [NEEDS CLARIFICATION: desired intel model — deterministic reveal vs. probabilistic/confidence-based results]
- **FR-008**: The system MUST prevent negative resources and enforce cost checks when deploying entities.
- **FR-009**: The system MUST persist match state so players can resume and audit past actions.
- **FR-010**: The system MUST provide an audit/log of resolved missions and agent investigations for replay and dispute resolution.

### Key Entities *(include if feature involves data)*

- **Player**: identity, chosen `Faction`, current resources, `Player Surface` reference, victory points.
- **Faction**: identifier, description, passive/active modifiers (e.g., stealth bonus, force bonus, economic bonus).
- **Player Surface**: container for the five zones (`Plans`, `PlayerMissions`, `OpponentMissions` (limited), `InvestigationZone`, `ResourcePool`).
- **Mission**: owner, cost (resource bundle), accumulated resources, state (planned/deployed/resolved), reward (victory points, side-effects), visibility rules.
- **Agent**: owner, location (deployed mission or Investigation zone), abilities (investigate, sabotage), cooldowns, cost.
- **Resource Pool**: per-player counts for `Credits`, `Force`, `Stealth`, `Investigation`, `Tech`.
- **GameSession**: match id, players, start timestamp, current tick, end condition, history of events.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Two players can create and complete a match where victory points are tallied and a winner is declared without data loss or corrupt state (testable by automated match simulation).
- **SC-002**: Resource regeneration (resource tick) operates according to the agreed cadence and players can spend regenerated resources to deploy entities (verifiable via tick logs and state snapshots).
- **SC-003**: Investigation results follow the specified intel model (deterministic or probabilistic) and can be validated by replaying events against ground-truth mission data.
- **SC-004**: The core gameplay loop (deploy mission/agent, accumulate resources, resolve missions) is executable end-to-end in an MVP and delivers a conclusive outcome for both players.

## Assumptions

- The system will initially target 2‑player matches only (no free-for-all or >2 players per match).
- "Daily" resource regeneration in the user description will be treated as a configurable resource tick; default cadence requires confirmation (see FR-005).
- The spec focuses on game mechanics and does not prescribe UI tech stacks or backend frameworks (those belong in `plan.md`).

## [NEEDS CLARIFICATION]

- Resource tick cadence and whether "daily" means once-per-calendar-day, once-per-24-hours, or a configurable faster cadence for testing.
- Match end condition: fixed duration vs first-to-X points vs player-agreed finish.
- Investigation intel model: deterministic full reveal vs probabilistic/confidence-based partial reveal.
