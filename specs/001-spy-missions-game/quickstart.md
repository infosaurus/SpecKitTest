# quickstart.md

## Quickstart (developer)

Prerequisites:

- .NET 8 SDK
- Node.js 18+
- PostgreSQL (for local development)

1. Restore and build backend/domain

```powershell
dotnet restore src/domain
dotnet build src/domain
dotnet restore src/backend
dotnet build src/backend
```

2. Run database locally (Postgres) and apply migrations (if using EF Core)

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
