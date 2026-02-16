# T001 — Monorepo Scaffold with Docker Compose

| Field | Value |
|-------|-------|
| Phase | 1 — Foundation |
| Agent Type | scaffold-agent |
| Depends On | — |
| Blocks | T002, T003, T004, T005, T006, T009, T012, T014, T016, T019, T025, T039 |
| Branch | `feat/T001-monorepo-scaffold` |
| Status | PENDING |

## Objective
Set up a monorepo workspace with four packages — server (Express+TypeScript), dashboard (React+Vite), shared (types+schemas), and extension (Chrome MV3 skeleton) — plus Docker Compose for local dev services (PostgreSQL, Redis, ChromaDB).

## Context
Everything depends on this scaffold. The monorepo structure defines the workspace boundaries, shared type system, and local development environment. Getting this right avoids painful restructuring later. Docker Compose ensures every contributor can spin up dependencies with a single command.

## Subtasks
1. Initialize root `package.json` with npm workspaces pointing to `server/`, `dashboard/`, `shared/`, `extension/`.
2. Create `tsconfig.base.json` with strict TypeScript settings and path aliases for `@agenttailor/shared`.
3. Scaffold `server/` — `package.json` (Express, TypeScript, ts-node-dev), `tsconfig.json` extending base, `src/index.ts` with health check endpoint on port 4000.
4. Scaffold `dashboard/` — `package.json`, `vite.config.ts` with React plugin and proxy to server, `tsconfig.json` extending base, `src/main.tsx` entry point, `index.html`, Tailwind CSS configuration.
5. Scaffold `shared/` — `package.json` with TypeScript compilation, `tsconfig.json` extending base, `src/index.ts` barrel export.
6. Scaffold `extension/` — `manifest.json` (Chrome Manifest V3), `background.ts`, `popup/popup.html`, `content/content.ts` stubs.
7. Write `docker-compose.yml` with PostgreSQL (port 5432), Redis (port 6379), and ChromaDB (port 8000) services with volumes for persistence.
8. Create `.env.example` with all required environment variables (DATABASE_URL, REDIS_URL, CHROMA_URL, CLERK_SECRET_KEY, OPENAI_API_KEY, etc.).
9. Create `.gitignore` covering node_modules, dist, .env, prisma client, coverage.
10. Configure `.eslintrc.cjs` and `.prettierrc` for consistent code style across all packages.
11. Add root scripts: `dev` (concurrently run server + dashboard), `build`, `lint`, `test`, `docker:up`, `docker:down`.

## Files to Create/Modify
- `package.json` — Root workspace config with scripts
- `tsconfig.base.json` — Shared TypeScript configuration
- `server/package.json` — Server dependencies (express, typescript, ts-node-dev, vitest)
- `server/tsconfig.json` — Server TS config extending base
- `server/src/index.ts` — Express app with health check endpoint
- `dashboard/package.json` — Dashboard dependencies (react, vite, tailwindcss)
- `dashboard/vite.config.ts` — Vite config with React plugin and API proxy
- `dashboard/tsconfig.json` — Dashboard TS config extending base
- `dashboard/src/main.tsx` — React entry point
- `dashboard/index.html` — HTML shell
- `shared/package.json` — Shared package config
- `shared/tsconfig.json` — Shared TS config extending base
- `shared/src/index.ts` — Barrel export
- `extension/manifest.json` — Chrome Manifest V3 definition
- `extension/background.ts` — Service worker stub
- `extension/popup/popup.html` — Extension popup shell
- `extension/content/content.ts` — Content script stub
- `docker-compose.yml` — PostgreSQL, Redis, ChromaDB services
- `.env.example` — Environment variable template
- `.gitignore` — Ignore patterns
- `.eslintrc.cjs` — ESLint configuration
- `.prettierrc` — Prettier configuration

## Acceptance Criteria
- [ ] `npm install` succeeds from root and installs all workspace dependencies
- [ ] `docker compose up -d` starts PostgreSQL, Redis, and ChromaDB without errors
- [ ] `npm run dev` starts both server (port 4000) and dashboard (port 5173)
- [ ] `GET /health` on server returns `{ status: "ok" }`
- [ ] Dashboard loads in browser and proxies API requests to server
- [ ] `shared` package can be imported from both server and dashboard
- [ ] TypeScript compilation succeeds across all packages with `npm run build`
- [ ] ESLint and Prettier run without errors on scaffolded code
- [ ] `.env.example` documents every required environment variable
