# Execution State

Current state of the AgentTailor project for session continuity. Updated after each phase execution.

---

## Current Status

- **Last completed phase**: Phase 1 — Foundation (8/8 tasks DONE)
- **Next phase**: Phase 2 — Context Intelligence Engine (7 tasks PENDING)
- **Branch**: `main` (all Phase 1 merged)
- **Typecheck**: Clean (`npm run typecheck` passes)
- **Working tree**: Clean (no uncommitted changes)
- **Code repo**: `/Users/ofek/Projects/Claude/AgentTailor/agenttailor`
- **Docs repo**: `/Users/ofek/Projects/Claude/AgentTailor/agenttailor-docs`

---

## Phase 1 Execution Summary

All 8 tasks completed across 6 batches:

| Batch | Tasks | Notes |
|-------|-------|-------|
| 1 | T001 (scaffold) | Monorepo, Docker Compose, npm workspaces |
| 2 | T002 (Prisma), T005 (chunking) | Ran in parallel |
| 3 | T003 (Clerk auth) | Fixed unused `clerkRequireAuth` import |
| 4 | T004 (Project CRUD), T006 (embeddings) | Fixed `req.params.id` undefined guards (3x) |
| 5 | T007 (document pipeline) | Created `pdf-parse.d.ts` for missing types |
| 6 | T008 (semantic search) | Clean typecheck on first try |

### Execution Pattern That Worked

Each task follows this pattern:
1. Launch agent on `feat/TXXX-task-name` branch
2. Agent creates files and commits
3. Main thread: `npm install` → `npm run typecheck` → fix errors → commit fixes
4. Merge to `main` → delete branch → next batch

**Sequential within batches** (not parallel) to avoid git branch contention.

### Recurring Issues & Fixes

| Issue | Cause | Fix Pattern |
|-------|-------|-------------|
| `req.params.id` is `string \| undefined` | `noUncheckedIndexedAccess: true` in tsconfig | Extract to const + null guard: `if (!id) return res.status(400)...` |
| Missing type declarations | Third-party packages without TS types (e.g., `pdf-parse`) | Create `server/src/types/{package}.d.ts` with `declare module` |
| Unused imports | Agent imports something it doesn't use | Remove the unused import |

---

## Current Codebase Inventory

### Shared Package (`shared/src/`)

Barrel export at `shared/src/index.ts` re-exports:
- `constants.ts` — App constants
- `schemas/chunk.ts` — ChunkSchema, ChunkMetadataSchema
- `schemas/user.ts` — UserSchema
- `schemas/project.ts` — CreateProjectInput, UpdateProjectInput, ProjectResponse, ProjectListResponse, ProjectListQuery
- `schemas/embedding.ts` — EmbeddingConfigSchema, EmbeddingResultSchema, VectorSearchResultSchema
- `schemas/search.ts` — SearchRequestSchema, SearchResultSchema, SearchResponseSchema, SuggestRequestSchema, SuggestResponseSchema

### Server Package (`server/src/`)

**Routes** (`routes/`):
- `index.ts` — Central router mounting: `/auth`, `/projects`, `/projects` (documents), `/search` + `/info` endpoint
- `auth.ts` — Auth routes (Clerk webhook, user sync)
- `projects.ts` — Project CRUD: POST/GET/PUT/DELETE with `authenticatedUser` middleware
- `documents.ts` — Document upload/list/get/delete under `/projects/:projectId/documents`
- `search.ts` — `POST /search/docs` (full search), `POST /search/suggest` (lightweight)

**Services** (`services/`):
- `projectService.ts` — Project CRUD with pagination, search, userId scoping
- `searchService.ts` — Embed query → vector search → Prisma hydrate → highlights
- `documentProcessor.ts` — 7-step pipeline: read → extract → chunk → save → embed → store → update status
- `chunking/` — Semantic chunker with heading-aware and paragraph-boundary strategies
- `embedding/embedder.ts` — OpenAI text-embedding-ada-002, batch splitting, retry with backoff
- `vectorStore/` — ChromaDB + Pinecone adapters behind `VectorStore` interface, factory pattern
- `textExtractor/` — PDF, DOCX, Markdown, code (26 extensions) extractors

**Middleware** (`middleware/`):
- `auth.ts` — `clerkAuth`, `requireAuth`, `attachUser`, `authenticatedUser`
- `errorHandler.ts` — Global error handler
- `validateRequest.ts` — Zod validation middleware

**Other**:
- `lib/db.ts`, `lib/prisma.ts` — Prisma client singleton
- `lib/openai.ts` — OpenAI client singleton (lazy init)
- `lib/queue.ts` — Bull queue for document processing
- `workers/documentWorker.ts` — Bull worker (3 concurrent jobs)
- `types/express.d.ts` — Express augmentation for Clerk auth
- `types/pdf-parse.d.ts` — Custom type declaration for pdf-parse

### Prisma Schema (6 models, 4 enums)

**Enums**: Plan (FREE/PRO/TEAM), DocumentType (PDF/DOCX/MARKDOWN/TXT/CODE), DocumentStatus (PROCESSING/READY/ERROR), TargetPlatform (CHATGPT/CLAUDE)

**Models**:
- `User` — id, email, name, clerkId, plan, settings
- `Project` — id, userId, name, description (→ User cascade)
- `Document` — id, projectId, filename, mimeType, type, status, chunkCount, sizeBytes, filePath (→ Project cascade)
- `Chunk` — id, documentId, content, position, metadata, tokenCount (→ Document cascade)
- `TailoringSession` — id, userId, projectId, taskInput, assembledContext, targetPlatform, tokenCount, qualityScore, metadata (→ User, Project cascade)
- `WebSearchResult` — id, sessionId, query, url, title, snippet, relevanceScore (→ TailoringSession cascade)

### Docker Compose Services
- PostgreSQL (port 5432)
- Redis (port 6379)
- ChromaDB (port 8000)

---

## Phase 2 Execution Plan

### Dependencies

```
T009 (Task Analyzer)       — depends on T001 ✓
T010 (Relevance Scorer)    — depends on T008 ✓
T011 (Gap Detector)        — depends on T010
T012 (Context Compressor)  — depends on T001 ✓
T013 (Source Synthesizer)   — depends on T010, T012
T014 (Context Window Mgr)  — depends on T001 ✓
T015 (Tailoring Endpoint)  — depends on T009, T010, T011, T012, T013, T014
```

### Computed Batches

```
Batch 1 (parallel): T009, T012, T014 — no intra-phase deps, all cross-phase deps met
Batch 2 (single):   T010 — depends on T008 (done), no intra-phase deps
Batch 3 (parallel): T011, T013 — depend on T010 (batch 2) and T012 (batch 1)
Batch 4 (single):   T015 — depends on ALL of T009-T014
```

All tasks use `intelligence-agent` (sonnet model).

### User-Provided Pipeline Context

The user provided specific design decisions for the intelligence pipeline:

```
Pipeline Flow:
  Task Analyzer → Retrieval → Reranking → Compression → Formatting → Gap Detection

Retrieval:
  - Top 20 candidates from vector DB (bi-encoder)
  - Cross-encoder reranking

Compression Thresholds:
  - > 0.8  → FULL (verbatim)
  - 0.5-0.8 → SUMMARY (LLM-summarized)
  - 0.3-0.5 → KEYWORDS (key terms only)
  - < 0.3  → DROP (exclude entirely)

Platform Formatting:
  - GPT:    Markdown sections
  - Claude: XML tags (<project_docs>, <web_research>, <task_analysis>)

Gap Detection:
  - < 60% coverage → triggers web search
```

**Important**: The task specs reference `<0.3 → REFERENCE` but the user corrected to `<0.3 → DROP`. The orchestrator/agent should follow the user's intent: chunks below 0.3 are excluded entirely, not included as references.

Also note: task spec T010 says `candidateCount: 30` but the user specified 20. Follow user's intent: **top 20 candidates**.

### New Environment Variables for Phase 2

```bash
# Cross-encoder (T010)
CROSS_ENCODER_PROVIDER="cohere"    # or "llm" for OpenAI/Claude fallback
COHERE_API_KEY=""
CROSS_ENCODER_MODEL=""

# Token counting (T014)
# Uses js-tiktoken (npm package) — no env vars needed
```

### New npm Dependencies Expected

| Package | Task | Purpose |
|---------|------|---------|
| `cohere-ai` | T010 | Cohere Rerank API for cross-encoder |
| `js-tiktoken` | T014 | Token counting (GPT-4/Claude tokenizer) |

### Files to Create (Phase 2)

**Shared schemas** (`shared/src/schemas/`):
- `taskAnalysis.ts` — T009
- `scoredChunk.ts` — T010
- `gapAnalysis.ts` — T011
- `compressedContext.ts` — T012
- `synthesizedContext.ts` — T013
- `tokenBudget.ts` — T014
- `tailor.ts` — T015

**Server intelligence services** (`server/src/services/intelligence/`):
- `domainClassifier.ts` — T009
- `taskAnalyzer.ts` — T009
- `crossEncoder.ts` — T010
- `relevanceScorer.ts` — T010
- `gapDetector.ts` — T011
- `summarizer.ts` — T012
- `contextCompressor.ts` — T012
- `priorityRanker.ts` — T013
- `sourceSynthesizer.ts` — T013
- `contextWindowManager.ts` — T014

**Server lib** (`server/src/lib/`):
- `tokenCounter.ts` — T014

**Server routes + orchestrator** (`server/src/`):
- `services/tailorOrchestrator.ts` — T015
- `services/platformFormatter.ts` — T015
- `routes/tailor.ts` — T015

**Barrel export updates**:
- `shared/src/index.ts` — Add all 7 new schema exports
- `server/src/routes/index.ts` — Mount `/tailor` routes

---

## Git History (Code Repo)

```
accfc44 [Phase 1] T008: Semantic search endpoint
3d35de5 [Phase 1] T007: Document upload and processing pipeline
22b23d4 [Phase 1] T006: Embedding generation and vector storage
3b61e94 [Phase 1] T004: Project CRUD API
e96beb8 [Phase 1] T003: Clerk auth integration
1aa4343 [Phase 1] T005: Semantic chunking engine
cb5fe47 [Phase 1] T002: PostgreSQL setup with Prisma ORM
fa7edaa [Phase 1] T001: Monorepo scaffold with Docker Compose
8034b98 Generate project-specific .claude/ config via /tailor-agents
9a73436 Initial scaffold
```

---

## Instructions for Next Session

1. Read this file first for full context
2. Run `/execute-phase 2` to start Phase 2
3. The batch plan is pre-computed above (4 batches)
4. Follow the same execution pattern: agent → install → typecheck → fix → commit → merge
5. Incorporate the user-provided pipeline context (score thresholds, formatting, candidate count)
6. After Phase 2 completes, update this file with Phase 2 results
