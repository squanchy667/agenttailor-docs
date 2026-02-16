# Task Board

## How This Works

Each task has a dedicated spec file in `tasks/` with full context for an autonomous agent to execute it independently. Tasks within a phase can run in parallel unless blocked by dependencies.

### Task Lifecycle
```
PENDING → IN_PROGRESS → IN_REVIEW → DONE
                ↓
            BLOCKED (waiting on dependency)
```

### Agent Assignment
Each task specifies a recommended agent type. When starting work, spin up an agent for each unblocked task and let them work in parallel.

---

## Phase 1 — Foundation

| ID | Task | Agent Type | Depends On | Status |
|----|------|-----------|------------|--------|
| T001 | [Monorepo scaffold with Docker Compose](tasks/phase-1/T001-monorepo-scaffold.md) | scaffold-agent | — | DONE |
| T002 | [PostgreSQL setup with Prisma ORM](tasks/phase-1/T002-postgresql-prisma-setup.md) | backend-agent | T001 | DONE |
| T003 | [Auth integration with Clerk](tasks/phase-1/T003-auth-integration.md) | backend-agent | T002 | DONE |
| T004 | [Project CRUD API](tasks/phase-1/T004-project-crud-api.md) | backend-agent | T002, T003 | DONE |
| T005 | [Semantic chunking engine](tasks/phase-1/T005-semantic-chunking-engine.md) | pipeline-agent | T001 | DONE |
| T006 | [Embedding generation and vector storage](tasks/phase-1/T006-embedding-vector-storage.md) | pipeline-agent | T002 | DONE |
| T007 | [Document upload and processing pipeline](tasks/phase-1/T007-document-processing-pipeline.md) | backend-agent | T004, T005, T006 | DONE |
| T008 | [Semantic search endpoint](tasks/phase-1/T008-semantic-search-endpoint.md) | backend-agent | T006 | DONE |

## Phase 2 — Context Intelligence Engine

| ID | Task | Agent Type | Depends On | Status |
|----|------|-----------|------------|--------|
| T009 | [Task Analyzer module](tasks/phase-2/T009-task-analyzer.md) | intelligence-agent | T001 | PENDING |
| T010 | [Relevance Scorer with cross-encoder](tasks/phase-2/T010-relevance-scorer.md) | intelligence-agent | T008 | PENDING |
| T011 | [Gap Detector](tasks/phase-2/T011-gap-detector.md) | intelligence-agent | T010 | PENDING |
| T012 | [Context Compressor](tasks/phase-2/T012-context-compressor.md) | intelligence-agent | T001 | PENDING |
| T013 | [Source Synthesizer and Priority Ranker](tasks/phase-2/T013-source-synthesizer.md) | intelligence-agent | T010, T012 | PENDING |
| T014 | [Context Window Manager](tasks/phase-2/T014-context-window-manager.md) | intelligence-agent | T001 | PENDING |
| T015 | [Tailoring orchestration endpoint](tasks/phase-2/T015-tailor-orchestration.md) | intelligence-agent | T009, T010, T011, T012, T013, T014 | PENDING |

## Phase 3 — Web Search Integration

| ID | Task | Agent Type | Depends On | Status |
|----|------|-----------|------------|--------|
| T016 | [Web search API client](tasks/phase-3/T016-web-search-client.md) | search-agent | T001 | PENDING |
| T017 | [Gap-triggered web search](tasks/phase-3/T017-gap-triggered-search.md) | search-agent | T011, T016 | PENDING |
| T018 | [Citation tracking and source attribution](tasks/phase-3/T018-citation-tracking.md) | search-agent | T017 | PENDING |

## Phase 4 — Dashboard Frontend

| ID | Task | Agent Type | Depends On | Status |
|----|------|-----------|------------|--------|
| T019 | [React dashboard scaffold with auth](tasks/phase-4/T019-dashboard-scaffold.md) | frontend-agent | T003 | PENDING |
| T020 | [Base UI components and layout](tasks/phase-4/T020-base-ui-components.md) | frontend-agent | T019 | PENDING |
| T021 | [Project management pages](tasks/phase-4/T021-project-management.md) | frontend-agent | T020, T004 | PENDING |
| T022 | [Document upload and processing status](tasks/phase-4/T022-document-upload-ui.md) | frontend-agent | T020, T007 | PENDING |
| T023 | [Tailoring session viewer](tasks/phase-4/T023-tailoring-session-viewer.md) | frontend-agent | T020, T015 | PENDING |
| T024 | [Settings and preferences page](tasks/phase-4/T024-settings-page.md) | frontend-agent | T020 | PENDING |

## Phase 5 — Browser Extension

| ID | Task | Agent Type | Depends On | Status |
|----|------|-----------|------------|--------|
| T025 | [Chrome Extension shell (Manifest V3)](tasks/phase-5/T025-extension-shell.md) | extension-agent | T001 | PENDING |
| T026 | [ChatGPT content script and detection](tasks/phase-5/T026-chatgpt-content-script.md) | extension-agent | T025 | PENDING |
| T027 | [Claude content script and detection](tasks/phase-5/T027-claude-content-script.md) | extension-agent | T025 | PENDING |
| T028 | [Side panel UI with context preview](tasks/phase-5/T028-side-panel-ui.md) | extension-agent | T025 | PENDING |
| T029 | [Context injection for both platforms](tasks/phase-5/T029-context-injection.md) | extension-agent | T026, T027, T028, T015 | PENDING |
| T030 | [Extension settings and project selection](tasks/phase-5/T030-extension-settings.md) | extension-agent | T025 | PENDING |

## Phase 6 — MCP Server

| ID | Task | Agent Type | Depends On | Status |
|----|------|-----------|------------|--------|
| T031 | [MCP server with tailor_context tool](tasks/phase-6/T031-mcp-server.md) | integration-agent | T015 | PENDING |
| T032 | [search_docs and upload_document tools](tasks/phase-6/T032-mcp-tools.md) | integration-agent | T031, T008, T007 | PENDING |
| T033 | [Resource providers for docs and sessions](tasks/phase-6/T033-mcp-resources.md) | integration-agent | T031 | PENDING |

## Phase 7 — Custom GPT

| ID | Task | Agent Type | Depends On | Status |
|----|------|-----------|------------|--------|
| T034 | [GPT Action schema and API adaptations](tasks/phase-7/T034-gpt-actions.md) | integration-agent | T015 | PENDING |
| T035 | [GPT Store listing preparation](tasks/phase-7/T035-gpt-store-listing.md) | integration-agent | T034 | PENDING |

## Phase 8 — Polish & Launch

| ID | Task | Agent Type | Depends On | Status |
|----|------|-----------|------------|--------|
| T036 | [Quality scoring system](tasks/phase-8/T036-quality-scoring.md) | polish-agent | T015 | PENDING |
| T037 | [Usage analytics dashboard](tasks/phase-8/T037-analytics-dashboard.md) | frontend-agent | T019, T036 | PENDING |
| T038 | [Rate limiting and plan enforcement](tasks/phase-8/T038-rate-limiting.md) | backend-agent | T003 | PENDING |
| T039 | [Landing page](tasks/phase-8/T039-landing-page.md) | frontend-agent | T001 | PENDING |
| T040 | [E2E test suite and production deployment](tasks/phase-8/T040-tests-and-deployment.md) | test-agent | T036, T037, T038, T039, T029, T035 | PENDING |

---

## Summary

| Phase | Theme | Tasks | Agent Types |
|-------|-------|-------|-------------|
| 1 | Foundation | 8 | scaffold, backend, pipeline |
| 2 | Intelligence Engine | 7 | intelligence |
| 3 | Web Search | 3 | search |
| 4 | Dashboard | 6 | frontend |
| 5 | Browser Extension | 6 | extension |
| 6 | MCP Server | 3 | integration |
| 7 | Custom GPT | 2 | integration |
| 8 | Polish & Launch | 5 | polish, frontend, backend, test |
| **Total** | | **40** | **9 agent types** |
