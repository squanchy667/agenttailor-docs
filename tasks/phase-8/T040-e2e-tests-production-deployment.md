# T040 — E2E Test Suite and Production Deployment

| Field | Value |
|-------|-------|
| Phase | 8 — Polish & Launch |
| Agent Type | test-agent |
| Depends On | T036, T037, T038, T039, T029, T035 |
| Blocks | — |
| Branch | `feat/T040-e2e-tests-production-deployment` |
| Status | PENDING |

## Objective
Write comprehensive tests across the stack (unit tests for schemas and services, integration tests for API endpoints, component tests for key dashboard UI), configure a CI pipeline for automated quality gates, and set up production deployment with Docker and cloud hosting configuration.

## Context
This is the final task before launch. Every feature from previous phases must be tested and deployable. The test suite serves as a safety net for future development — without it, any change risks breaking existing functionality. The CI pipeline ensures that no broken code reaches production. Docker containerization and cloud deployment config make the difference between "it works on my machine" and "it works for users". This task ties everything together into a production-ready system.

## Subtasks
1. Create `server/vitest.config.ts` — Vitest configuration for the server package:
   - Test environment: node.
   - Coverage: v8 provider, thresholds at 70% lines.
   - Setup file for test database and mocks.
   - Path aliases matching tsconfig.
2. Create `dashboard/vitest.config.ts` — Vitest configuration for the dashboard:
   - Test environment: jsdom.
   - Setup file with React Testing Library configuration.
   - CSS module mocks.
3. Write schema unit tests:
   - `shared/src/schemas/__tests__/project.test.ts` — Test CreateProjectInput validation (valid input, empty name, name too long, optional description). Test ProjectResponse parsing.
   - `shared/src/schemas/__tests__/tailor.test.ts` — Test TailorRequest validation (valid input, missing task, invalid projectId). Test TailorResponse parsing with quality scores.
4. Write service unit tests:
   - `server/src/services/__tests__/tailorOrchestrator.test.ts` — Mock dependencies (searchService, webSearch, qualityScorer). Test full pipeline: document search + web search + assembly + scoring. Test fallback when web search fails. Test quality score inclusion.
   - `server/src/services/__tests__/searchService.test.ts` — Mock ChromaDB/Pinecone client. Test semantic search returns ranked results. Test empty results for unmatched queries. Test topK and minScore filtering.
5. Write API integration tests:
   - `server/src/routes/__tests__/projects.test.ts` — Test CRUD lifecycle: create, read, list (with pagination), update, delete. Test auth enforcement (401 without token). Test validation errors (400). Test cross-user isolation.
   - `server/src/routes/__tests__/tailor.test.ts` — Test tailoring endpoint with mocked services. Test rate limiting (429 after exceeding limit). Test quality score in response. Test missing project (404).
6. Write component tests:
   - `dashboard/src/components/__tests__/ProjectCard.test.tsx` — Test renders project name, description, document count. Test click navigates to project detail. Test delete confirmation dialog.
   - `dashboard/src/components/__tests__/ContextViewer.test.tsx` — Test renders assembled context. Test source attribution display. Test quality score badge. Test copy-to-clipboard functionality.
7. Create `Dockerfile` — Multi-stage Docker build:
   - Stage 1 (deps): Install all workspace dependencies.
   - Stage 2 (build): Build server, shared, dashboard, and landing packages.
   - Stage 3 (production): Copy only built artifacts, install production dependencies, expose port 4000.
   - Use node:20-alpine as base image.
   - Include health check instruction.
8. Create `docker-compose.prod.yml` — Production Docker Compose:
   - Services: app (built from Dockerfile), postgres (with volume), redis (with volume), chromadb.
   - Environment variables from `.env.production`.
   - Restart policy: unless-stopped.
   - Network isolation between services.
9. Create `.github/workflows/ci.yml` — GitHub Actions CI pipeline:
   - Trigger: push to main, pull requests.
   - Jobs: lint (eslint), typecheck (tsc --noEmit across all packages), test (vitest run with coverage), build (docker build).
   - PostgreSQL and Redis service containers for integration tests.
   - Cache node_modules and Docker layers.
   - Fail-fast: stop all jobs if any fails.
10. Create deployment config — `railway.toml` (Railway) or `render.yaml` (Render):
    - Service definitions for web app, PostgreSQL, Redis.
    - Build command, start command, health check path.
    - Environment variable references.
    - Auto-deploy from main branch.

## Files to Create/Modify
- `server/vitest.config.ts` — Server test configuration
- `dashboard/vitest.config.ts` — Dashboard test configuration
- `shared/src/schemas/__tests__/project.test.ts` — Project schema unit tests
- `shared/src/schemas/__tests__/tailor.test.ts` — Tailor schema unit tests
- `server/src/services/__tests__/tailorOrchestrator.test.ts` — Orchestrator service tests
- `server/src/services/__tests__/searchService.test.ts` — Search service tests
- `server/src/routes/__tests__/projects.test.ts` — Projects API integration tests
- `server/src/routes/__tests__/tailor.test.ts` — Tailor API integration tests
- `dashboard/src/components/__tests__/ProjectCard.test.tsx` — ProjectCard component tests
- `dashboard/src/components/__tests__/ContextViewer.test.tsx` — ContextViewer component tests
- `Dockerfile` — Multi-stage production Docker build
- `docker-compose.prod.yml` — Production Docker Compose configuration
- `.github/workflows/ci.yml` — GitHub Actions CI pipeline
- `railway.toml` — Railway deployment configuration (or `render.yaml`)

## Acceptance Criteria
- [ ] All schema unit tests pass — valid inputs accepted, invalid inputs rejected with correct errors
- [ ] Service unit tests pass with mocked dependencies, covering happy paths and error cases
- [ ] Integration tests pass for project CRUD lifecycle including auth and validation
- [ ] Integration tests pass for tailoring endpoint including rate limiting scenarios
- [ ] Component tests pass for ProjectCard (render, navigation, delete) and ContextViewer (render, sources, copy)
- [ ] Test coverage exceeds 70% line coverage for server and shared packages
- [ ] CI pipeline runs lint, typecheck, test, and build steps on push and PR
- [ ] CI pipeline uses service containers for PostgreSQL and Redis in test jobs
- [ ] `docker build .` succeeds and produces a working container image
- [ ] `docker compose -f docker-compose.prod.yml up` starts all services and app is accessible
- [ ] Deployment config is valid and references all required services and environment variables
- [ ] Health check endpoint responds in the Docker container
