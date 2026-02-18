# Changelog

All notable changes to AgentTailor will be documented in this file.

## [Unreleased]

<!-- Entries added automatically by /sync-docs and /execute-phase -->

## [2026-02-18] v1.1 — Local Mode (Plug-and-Play)

Zero API keys, zero accounts. `git clone → npm run setup → npm run dev`.

### Batch 1 — Foundation
- T041: Made Prisma `clerkId` optional, updated Zod schemas for local user support — DONE
- T046: Fixed MCP server default port (3000 → 4000) to match server — DONE
- T047: Created `.env.local.example` and added AUTH_MODE/EMBEDDING_PROVIDER to `.env.example` — DONE
- T049: Added NoopCrossEncoder for local mode (no external reranking API needed) — DONE

### Batch 2 — Core Implementations
- T042: Created local auth middleware (auto-created PRO user, no Clerk dependency) — DONE
- T044: Added local embedder with `@xenova/transformers` (all-MiniLM-L6-v2, 384 dims) — DONE
- T048: Setup script (`npm run setup`) with seed data and Docker orchestration — DONE
- T050: Softened MCP server API key warning for local mode — DONE

### Batch 3 — Wiring
- T043: Created auth mode router with dynamic imports (AUTH_MODE=local|clerk) — DONE
- T045: Made embedding config defaults dynamic based on EMBEDDING_PROVIDER — DONE

### Batch 4 — Polish
- T051: Updated README with local quickstart and MCP config snippet — DONE
- T052: Added local mode smoke tests (auth, embedder, cross-encoder factories) — DONE

### Additional Fixes (discovered during local testing)
- Dashboard dual-mode auth abstraction (`authProvider.tsx`) — no Clerk keys needed in local mode
- Document worker startup (was defined but never called)
- Docker port conflict resolution (postgres 5433, chromadb 8100)
- Prisma cuid compatibility (replaced `.uuid()` with `.min(1)` in Zod schemas)
- 162 tests passing, 10 documents processed into 60 chunks end-to-end

## [2026-02-18] Phase 8 — Polish & Launch

### Batch 1
- T036: Quality scoring system with sub-scores (coverage, diversity, relevance, compression) and suggestions — DONE
- T039: Landing page with hero, features, pricing, and footer components — DONE

### Batch 2
- T037: Usage analytics dashboard with Recharts, session/quality trends, project stats, plan usage — DONE
- T038: Rate limiting and plan enforcement with Redis (FREE 50/day, PRO 500/day, TEAM 2000/day) — DONE

### Batch 3
- T040: E2E tests (38 passing), Dockerfile, docker-compose.prod.yml, CI pipeline, Railway config — DONE

## [2026-02-18] Phase 7 — Custom GPT

### Batch 1
- T034: GPT Action schema with OpenAPI 3.1 spec, API key auth, and /gpt/ endpoints — DONE

### Batch 2
- T035: GPT Store listing with system instructions, conversation starters, and privacy policy — DONE

## [2026-02-17] Phase 6 — MCP Server

### Batch 1
- T031: MCP server with tailor_context tool and stdio transport — DONE

### Batch 2
- T032: search_docs, upload_document, and list_projects MCP tools — DONE
- T033: Resource providers for documents and sessions — DONE

## [2026-02-17] Phase 5 — Browser Extension

### Batch 1
- T025: Chrome extension shell with Manifest V3 and side panel — DONE

### Batch 2
- T026: ChatGPT content script with ProseMirror detection and injection — DONE
- T027: Claude content script with contenteditable detection and injection — DONE
- T028: Side panel UI with context preview and source list — DONE
- T030: Extension settings with project selector and storage wrapper — DONE

### Batch 3
- T029: Context injection flow with tailor bridge and auto-tailor mode — DONE

## [2026-02-17] Phase 4 — Dashboard Frontend

### Batch 1
- T019: React dashboard scaffold with Clerk auth flow — DONE

### Batch 2
- T020: Base UI components, API client, and React Query setup — DONE

### Batch 3
- T021: Project management pages with CRUD and detail view — DONE
- T022: Document upload UI with drag-and-drop and processing status — DONE
- T023: Tailoring session viewer with context display and history — DONE
- T024: Settings page with API key management and context preferences — DONE

## [2026-02-17] Phase 3 — Web Search Integration

### Batch 1
- T016: Web search API client with Tavily and Brave providers — DONE

### Batch 2
- T017: Gap-triggered web search with result processing — DONE

### Batch 3
- T018: Citation tracking with source attribution — DONE

## [2026-02-17] Phase 2 — Context Intelligence Engine

### Batch 1
- T009: Task analyzer module with domain classification — DONE
- T010: Relevance scorer with cross-encoder reranking — DONE
- T012: Context compressor with hierarchical compression — DONE
- T014: Context window manager with token budgeting — DONE

### Batch 2
- T011: Gap detector with coverage analysis — DONE
- T013: Source synthesizer with dedup and priority ranking — DONE

### Batch 3
- T015: Tailoring orchestration endpoint with full pipeline — DONE

## [2026-02-17] Phase 1 — Foundation

### Batch 1
- T001: Monorepo scaffold with Docker Compose — DONE

### Batch 2
- T002: PostgreSQL setup with Prisma ORM — DONE
- T005: Semantic chunking engine — DONE

### Batch 3
- T003: Auth integration with Clerk — DONE

### Batch 4
- T004: Project CRUD API — DONE
- T006: Embedding generation and vector storage — DONE

### Batch 5
- T007: Document upload and processing pipeline — DONE

### Batch 6
- T008: Semantic search endpoint — DONE
