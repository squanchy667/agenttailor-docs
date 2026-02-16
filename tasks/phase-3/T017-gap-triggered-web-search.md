# T017 — Gap-Triggered Web Search and Result Processing

| Field | Value |
|-------|-------|
| Phase | 3 — Web Search Integration |
| Agent Type | search-agent |
| Depends On | T011, T016 |
| Blocks | T018 |
| Branch | `feat/T017-gap-triggered-web-search` |
| Status | PENDING |

## Objective
Integrate web search into the Gap Detector pipeline so that when knowledge gaps are detected in the user's document base, the system automatically triggers targeted web searches, processes the results (fetches URL content, extracts text, creates temporary chunks), and merges them into the context assembly.

## Context
The Gap Detector (T011) identifies topics the user's docs don't cover well. Without web search integration, the system can only say "I don't have enough context." With this task, AgentTailor becomes proactive — it searches the web to fill gaps, processes the raw web content into embeddable chunks, and weaves them alongside document-sourced chunks. This is what makes AgentTailor dramatically better than plain RAG: it doesn't just retrieve from your docs, it goes and finds what's missing.

## Subtasks
1. Create `server/src/services/search/webResultProcessor.ts` — given a `WebSearchResult`, fetch the full page content via HTTP, extract meaningful text (strip nav, ads, boilerplate using a readability algorithm or mozilla/readability), split into chunks following the same strategy as the document chunker (T008), and generate embeddings for each chunk.
2. Define a `WebChunk` type that extends the base `Chunk` type with web-specific metadata (sourceUrl, fetchedAt, isTemporary: true, searchQuery that triggered it).
3. Modify `server/src/services/intelligence/gapDetector.ts` — after identifying gaps, generate search queries from gap descriptions (e.g., gap "missing OAuth2 flow details" becomes query "OAuth2 authorization code flow implementation"). Add a `generateSearchQueries(gaps: Gap[]): string[]` method.
4. Modify `server/src/services/tailorOrchestrator.ts` — after gap detection, check if web search is enabled for the project/user. If yes, execute web searches for each gap query, process results through `webResultProcessor`, embed the temporary chunks, run similarity search against them, and merge top web-sourced chunks into the context assembly with lower default weight than doc-sourced chunks.
5. Add configuration options: `webSearchEnabled` (boolean), `maxWebResults` (number), `webChunkWeight` (0-1 multiplier applied to relevance scores of web-sourced chunks).
6. Implement rate limiting and caching for web fetches — cache fetched URL content in Redis with 1-hour TTL to avoid re-fetching the same page within a session.
7. Write tests for query generation from gaps, web result processing, and the integrated pipeline flow.

## Files to Create/Modify
- `server/src/services/search/webResultProcessor.ts` — Fetch, extract, chunk, and embed web page content
- `shared/src/schemas/webSearch.ts` — Add WebChunk type extending base Chunk
- `server/src/services/intelligence/gapDetector.ts` — Modify: add search query generation from gaps
- `server/src/services/tailorOrchestrator.ts` — Modify: integrate web search results into context pipeline
- `server/src/services/search/webContentCache.ts` — Redis-backed cache for fetched web content
- `server/src/__tests__/services/search/webResultProcessor.test.ts` — Unit tests
- `server/src/__tests__/services/search/gapSearchIntegration.test.ts` — Integration tests

## Acceptance Criteria
- [ ] Web result processor fetches URL content and extracts clean text (no HTML tags, nav, ads)
- [ ] Extracted web content is chunked using the same strategy as document chunks
- [ ] Web chunks include sourceUrl, fetchedAt, isTemporary flag, and originating search query
- [ ] Gap detector generates relevant search queries from gap descriptions
- [ ] Tailor orchestrator triggers web search when gaps are detected and web search is enabled
- [ ] Web-sourced chunks are merged into context assembly with configurable weight multiplier
- [ ] Fetched URL content is cached in Redis with 1-hour TTL
- [ ] Duplicate URLs across multiple gap searches are fetched only once
- [ ] Pipeline gracefully handles web fetch failures (timeout, 404, blocked) without crashing
- [ ] All tests pass with `npm run test`
