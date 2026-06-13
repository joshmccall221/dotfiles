# CLAUDE.md — Incident Operations Apps

> Project-level context for the DT-SIP-INCOPS-APPS monorepo.
> Global developer context (devices, workflow, behavior) lives at ~/.claude/CLAUDE.md.
> This file adds project-specific architecture, conventions, and commands.

---

## What This Is

A monorepo with two applications (**Action Log** and **COP Tool**) sharing a
single .NET 8 backend host and a single React/TypeScript frontend (Portal).

---

## Architecture

### Backend (.NET 8)

```
Source/IncidentOperations/
├── Host/           → Unified ASP.NET Core entry point (Program.cs)
├── ActionLog/      → Action Log domain (controllers, services, repos)
├── COPTool/        → COP Tool domain (controllers, services, repos)
├── Common/         → Shared infrastructure (Kusto client, middleware, Azure clients)
├── Shared/         → Shared models/contracts (no dependencies)
├── Migration/      → Database migration tooling
└── Portal/         → React frontend (.esproj, included in solution)
```

- **Host** is the only runnable project — pulls in ActionLog and COPTool
  controllers via `AddApplicationPart(...)` and services via
  `AddActionLogServices()` / `AddCOPToolServices()` extension methods.
- Each domain library: `Controllers/ → Services/ → Repositories/` with
  interfaces in `Interfaces/` and DI registration in `Extensions/BuilderExtensions.cs`.
- Primary datastore: **Azure Data Explorer (Kusto)** — queries in KQL, not SQL.
- Auth: **MISE middleware** + Entra ID app-role authorization policies.

### Frontend (React + TypeScript + Vite)

```
Portal/src/
├── common/         → Shared auth, state, components, Kusto client, hooks
├── features/       → Feature-based slices (actionLog, copTool, kustoExplorer…)
│   └── <feature>/  → components/, pages/, api/, hooks/, models/, state/, index.ts
├── routes/         → Central routing (AppRoutes.tsx)
└── App.tsx / main.tsx
```

- **Feature-based architecture** — each tool owns a self-contained folder.
- State: **Redux Toolkit** with per-feature slices, RTK Query for API calls.
- UI: **Fluent UI v8/v9** + **Coherence Design System**.
- Frontend can query Kusto directly (browser → Kusto) for reads via `common/kusto/`.
- Graph visualization: **KeyLines**.

---

## Build, Test & Lint Commands

### Backend

```bash
cd Source/IncidentOperations

dotnet restore
dotnet build Host/Host.csproj

# Run API locally (https://localhost:5000)
ASPNETCORE_ENVIRONMENT=LOCAL dotnet run --project Host/Host.csproj
```

### Frontend

```bash
cd Source/IncidentOperations/Portal

npm install                  # standard install
npm run authinstall          # use this for Azure DevOps private feed auth
npm run dev                  # dev server → http://localhost:5173
npm run build                # tsc + vite build (production)
npm run lint                 # ESLint
```

---

## Key Conventions

### Adding a New Backend Feature

1. Add model in `Models/`, interface in `Interfaces/`
2. Implement service in `Services/`, repository in `Repositories/`
3. Add controller in `Controllers/`
4. Register in `Extensions/BuilderExtensions.cs`:
   - Scoped for services/repos
   - Singleton for stateless utilities

### Adding a New Frontend Feature

1. Create `features/<newTool>/` with subdirs:
   `components/`, `pages/`, `api/`, `hooks/`, `models/`, `state/`, `index.ts`
2. Create Redux slice in `state/`, register in `common/state/store.ts`
3. Add routes in `routes/AppRoutes.tsx` and nav entry in sidebar

### Naming Conventions

- **Backend (C#):** PascalCase folders — `Controllers/`, `Services/`, `Repositories/`
- **Frontend:** camelCase for utility folders (`hooks/`, `services/`),
  PascalCase for component folders (`IncidentGrid/`, `ShareDialog/`)
- Feature folders use barrel exports (`index.ts`) as public API

### Private Feeds

- **NuGet:** private Azure DevOps feed (`DT-SIP-IncidentOps`) — see `NuGet.config`
- **NPM:** private feed requiring credential auth — use `npm run authinstall`

### Git Branching

- Primary branches: `main` and `develop`
- Full branching strategy and PR standards: `Documentation/Project_Standards.md` §16
- Never commit directly to `main` or `develop`

---

## Project-Specific Claude Behavior

### Architecture decisions to flag before touching:
- Changes to `Host/Program.cs` or `BuilderExtensions.cs` — affect entire app
- Auth middleware (MISE) or Entra ID policy changes
- Any new Kusto query patterns — confirm KQL approach before writing
- New Redux slices — confirm state shape before implementing
- KeyLines graph changes — confirm data model first

### When adding features, always ask:
- Backend or frontend or both?
- Which domain — ActionLog or COPTool or shared Common?
- Does this need a new Kusto table/query or reuse existing?
- Does this need new app roles / auth policies?

### Test approach for this project:
- Backend: integration tests preferred over unit tests for repository layer
  (Kusto queries need to be tested against real or mock data)
- Frontend: Playwright for integration testing (already configured)
- Always run `npm run lint` before considering frontend work done

### Kusto / KQL reminders:
- This is not SQL — don't generate SQL syntax
- Queries go in `Repositories/` on the backend or `common/kusto/` on the frontend
- Direct browser → Kusto queries are read-only by design
- Ask about existing query patterns before writing new ones from scratch
