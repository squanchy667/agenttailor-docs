# T008 — Semantic Search Endpoint

| Field | Value |
|-------|-------|
| Phase | 1 — Foundation |
| Agent Type | backend-agent |
| Depends On | T006 |
| Blocks | T010, T032 |
| Branch | `feat/T008-semantic-search-endpoint` |
| Status | PENDING |

## Objective
Implement semantic search over project documents — query embedding, vector similarity search, metadata filtering, and result ranking. Expose as `POST /api/search/docs` for use by the tailoring pipeline and dashboard.

## Context
Semantic search is the retrieval backbone of AgentTailor. When the tailoring engine needs to find relevant context for a user's task, it calls this search endpoint. Unlike keyword search, semantic search understands meaning — "authentication middleware" will match chunks about "auth guards" and "JWT verification." This endpoint also serves the dashboard's document search UI.

## Subtasks
1. Create `shared/src/schemas/search.ts` — Zod schemas:
   - `SearchRequest` — query (string, 1-1000 chars), projectId (UUID), topK (number, default 10, max 50), filters (optional object: documentIds, documentTypes, minScore).
   - `SearchResult` — chunkId, documentId, documentFilename, content, score (0-1), metadata (headings, pageNum, section), highlights (relevant snippet).
   - `SearchResponse` — results array, totalFound, queryTokenCount, searchTimeMs.
2. Create `server/src/services/searchService.ts`:
   - `searchDocuments(query, projectId, options)`:
     a. Embed the query text using the Embedder.
     b. Query the vector store for top-K similar chunks in the project's collection.
     c. Fetch chunk records from PostgreSQL to get full content and document info.
     d. Apply post-filters (document type, minimum score threshold).
     e. Generate highlight snippets — extract the most relevant portion of each chunk.
     f. Return ranked results with scores, metadata, and highlights.
   - `searchMultiProject(query, projectIds, options)` — Search across multiple projects (for future multi-project context).
3. Create `server/src/routes/search.ts` — Express router:
   - `POST /api/search/docs` — Accept search request, validate with Zod, call searchService, return results. Include timing info in response.
   - `POST /api/search/suggest` — Lightweight endpoint that returns just chunk IDs and scores (for internal pipeline use, skips full content hydration).
4. Mount search router in `server/src/routes/index.ts`.
5. Add search-specific error handling: empty project (no documents indexed), embedding service unavailable, vector store timeout.

## Files to Create/Modify
- `server/src/routes/search.ts` — Search API endpoints
- `server/src/services/searchService.ts` — Search orchestration with embedding, vector query, and result hydration
- `shared/src/schemas/search.ts` — Zod schemas for search request, results, and response
- `server/src/routes/index.ts` — Mount search router

## Acceptance Criteria
- [ ] `POST /api/search/docs` returns relevant chunks ranked by similarity score
- [ ] Query "database migration patterns" finds chunks about "Prisma migrations" and "schema changes"
- [ ] Results include document filename, chunk content, similarity score, and metadata
- [ ] `topK` parameter limits the number of results returned
- [ ] `filters.documentIds` restricts search to specific documents
- [ ] `filters.minScore` filters out low-relevance results
- [ ] Highlight snippets show the most relevant portion of each chunk
- [ ] Response includes searchTimeMs for performance monitoring
- [ ] Searching an empty project returns an empty results array (not an error)
- [ ] Invalid projectId returns 404
- [ ] Request validation rejects empty queries and invalid UUIDs
- [ ] Search across multiple projects returns results tagged with project source
