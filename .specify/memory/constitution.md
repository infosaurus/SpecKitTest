<!--
Sync Impact Report

- Version change: 1.0.0 -> 1.1.0
- Modified principles:
	- Library-First (established)
	- Interface Contracts (CLI & APIs)
	- Test-First (NON-NEGOTIABLE)
	- Integration & Contract Testing
	- Observability, Versioning & Simplicity
	- (added) Architecture Style: Onion Architecture
	- (added) CQRS: separate read/write models
	- (added) Domain-Driven Design (tactical patterns: Aggregates)
	- (added) Domain Model Fidelity: use specs' domain concepts only
- Added sections: Development Workflow, Additional Constraints
- Removed sections: none
- Templates requiring updates:
	- .specify/templates/plan-template.md ✅ updated
	- .specify/templates/spec-template.md ✅ updated
	- .specify/templates/tasks-template.md ✅ updated
	- .specify/templates/checklist-template.md ✅ updated
	- .specify/templates/agent-file-template.md ⚠ pending (manual review recommended)
- Follow-up TODOs:
	- TODO(RATIFICATION_DATE): confirm original ratification date if different from 2026-01-13

-->

# SpecKitTest Constitution

## Core Principles

### I. Library-First
All new features MUST be designed as self-contained libraries or modules.
- Libraries MUST have clear public interfaces, documentation, and automated unit tests.
- Rationale: promotes reuse, independent testing, and clearer ownership boundaries.

### II. Interface Contracts (CLI & APIs)
All public functionality MUST expose a documented contract (CLI, HTTP API, or gRPC).
- Contracts MUST include input/output formats, error semantics, and versioning expectations.
- Rationale: explicit contracts reduce integration errors and make upgrades safer.

### III. Test-First (NON-NEGOTIABLE)
Tests MUST be written before (or alongside) implementation for every new unit of work.
- Unit tests MUST be automated and runnable locally; integration/contract tests MUST exist
	for cross-service boundaries or public interfaces.
- Rationale: ensures correctness, prevents regressions, and documents expected behavior.

### IV. Integration & Contract Testing
Changes that touch service boundaries, shared schemas, or public contracts MUST include
integration and contract tests that validate end-to-end behavior.
- Rationale: integration tests catch system-level regressions that unit tests cannot.

### V. Architecture Style — Onion Architecture
The project SHOULD follow the Onion Architecture to enforce clear separation between domain, application, and infrastructure concerns.
- The **Domain** layer MUST contain business logic, entities, value objects, and domain services and MUST NOT depend on infrastructure or UI.
- The **Application** layer (use-cases) MAY orchestrate domain operations and define interfaces (ports) for persistence or external services.
- The **Infrastructure** layer MUST implement the interfaces (adapters) and be replaceable without changing domain code.
- Rationale: Onion Architecture reduces coupling, enables testability of business rules, and supports the Library-First principle.

### VI. CQRS (Command Query Responsibility Segregation)
Designs SHOULD separate write models (commands/aggregates) from read models (projections/queries).
- Commands that mutate state MUST go through the domain/application write side; read requests SHOULD use optimized read models or projections.
- The system MUST provide consistency guarantees and document whether reads are eventually consistent or strongly consistent for each API.
- Rationale: CQRS clarifies responsibilities, improves scalability for read-heavy workloads, and aligns with DDD tactical patterns.

### VII. Domain-Driven Design (Tactical Patterns)
Teams MUST respect DDD tactical patterns where domain complexity warrants it.
- Aggregates: group related entities and value objects behind an aggregate root that enforces invariants and transactional boundaries.
- Entities and Value Objects: model identity semantics explicitly; use value objects for immutable concepts.
- Domain Events: prefer explicit domain events to communicate state changes across bounded contexts.
- Rationale: DDD tactical patterns help manage complexity, make invariants explicit, and improve maintainability for complex domains.

### VIII. Domain Model Fidelity
The project MUST only use domain concepts explicitly defined in the feature specification files under `/specs/*`.
- Engineers and contributors MUST NOT invent or infer new domain entity names, concepts, or types that are not present in the applicable `spec.md` and companion data model files.
- If a new domain type or concept is required, the implementer MUST open an explicit clarification with the spec owner and obtain approval before introducing it into the domain layer.
- Tooling and agents (bots, code generators) working on the repository MUST follow this rule and prompt for approval if they encounter a need to create new domain concepts.
- Rationale: Ensures a single source of truth for domain language, prevents accidental drift between spec and implementation, and preserves testability and governance of domain models.

## Additional Constraints
Security & Compliance:
- Sensitive data MUST be handled according to applicable law and best practices; secrets
	MUST NOT be committed to source control.

Performance Goals:
- Projects SHOULD document performance targets in `plan.md` when applicable.

Technology Constraints:
- Preferred languages, runtime versions, and primary tools SHOULD be recorded in each
	feature `plan.md`. If unspecified, default to pragmatic modern toolchains.

## Governance
Amendments:
- Amendments to this constitution require a documented proposal and approval by at
	least two maintainers or a majority of the core team, and MUST include a migration
	plan for material changes.

Compliance & Reviews:
- Periodic reviews (at least annually) SHOULD be scheduled to verify ongoing
	compliance with these principles.

**Version**: 1.2.0 | **Ratified**: 2026-01-13 | **Last Amended**: 2026-01-15

