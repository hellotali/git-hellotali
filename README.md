# hellotali – AI-Assisted Mood and Cognitive Monitoring (MVP)

An MVP for AI-powered mood and cognitive monitoring to support mental health and dementia care. Frontend in Angular 18 (PWA). Backend in .NET 8 Web API with Clean Architecture. OpenAI as default AI analyzer. PostgreSQL in all environments.

## Features (MVP)
- Daily text check-in (voice UI stub)
- Immediate AI mood analysis (OpenAI)
- Trend visualization and basic risk flags
- Secure, extensible API and DB foundation

## Architecture
- Frontend: Angular 18, Material, PWA
- Backend: .NET 8, EF Core, PostgreSQL, Swagger
- AI: OpenAI (default), optional Azure Text Analytics, local fallback

See `docs/architecture.md` for the diagram and Mermaid source.

## Quickstart
Prereqs: .NET 8 SDK, Node 18+, npm, Docker Desktop, OpenAI API key

1) Start PostgreSQL (Docker)
```
docker compose -f infra/docker-compose.postgres.yml up -d
```
2) Backend
```
cd backend/HelloTali.Api
setx DATABASE_URL "Host=localhost;Port=5432;Database=hellotali;Username=dev;Password=devpass"
setx AI_PROVIDER openai
setx OPENAI_API_KEY <your_key>
dotnet run
```
Swagger: `https://localhost:5001/swagger`

3) Frontend
```
cd frontend/hellotali-app
npm install
npm start
```
Dev server: `http://localhost:4200`

## Monorepo Structure
```
backend/   # .NET API and libraries
frontend/  # Angular app
docs/      # planning and architecture docs
infra/     # docker-compose and ops files
```

## Environments
- development, staging, production
- Backend env vars: `DATABASE_URL`, `AI_PROVIDER=openai`, `OPENAI_API_KEY`, optional `OPENAI_MODEL`
- Frontend config: `API_BASE_URL`

## Docs
- Plan: `docs/app-plan.md`
- Procedures: `docs/procedure.md`
- Architecture: `docs/architecture.md`

## License
TBD

