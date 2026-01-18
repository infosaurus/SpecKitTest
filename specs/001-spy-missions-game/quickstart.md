# quickstart.md

## Quickstart (developer)

Prerequisites:

- .NET 8 SDK
- Node.js 18+

1. Restore and build backend/domain

```powershell
dotnet restore src/domain
dotnet build src/domain
dotnet restore src/backend
dotnet build src/backend
```

2. Configure file-based storage

- The MVP uses a simple file-based event store and JSON read-models under the `data/` directory at repository root. Ensure the backend process has write access to `./data`.
- Default layout (created automatically by the backend on first run):
	- `./data/events/` — per-match append-only event files (`{matchId}.events`)
	- `./data/snapshots/` — optional aggregate snapshots (`{matchId}.snapshot.json`)
	- `./data/read-models/games/` — JSON read-models (`{matchId}.json`)
	- `./data/idempotency/` — idempotency key mappings

3. Run backend

```powershell
cd src/backend
dotnet run --urls http://localhost:5000
```

4. Run frontend

```bash
cd frontend
npm install
npm run start
```

5. Test `POST /api/v1/games` using the provided OpenAPI contract at `/specs/001-spy-missions-game/contracts/openapi.yaml`.

Notes:

- This quickstart uses a file-based persistence model to avoid RDBMS dependency for MVP. For production or higher scale, migrate the persistence layer to an appropriate store and update the infrastructure adapters.
