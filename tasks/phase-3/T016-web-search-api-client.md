# T016 — Web Search API Client (Tavily/Brave)

| Field | Value |
|-------|-------|
| Phase | 3 — Web Search Integration |
| Agent Type | search-agent |
| Depends On | T001 |
| Blocks | T017 |
| Branch | `feat/T016-web-search-api-client` |
| Status | PENDING |

## Objective
Implement a web search module with Tavily API as the primary provider and Brave Search as a fallback. The module returns structured search results with title, URL, snippet, and relevance metadata through a unified interface.

## Context
AgentTailor needs to supplement documentation-based context with live web knowledge. When a user's task touches topics not covered by their uploaded docs, web search fills the gap. A dual-provider setup with Tavily (optimized for AI retrieval) and Brave (privacy-focused fallback) ensures reliability. This module is the foundation that the gap-triggered search pipeline (T017) and citation tracking (T018) build upon.

## Subtasks
1. Define Zod schemas in `shared/src/schemas/webSearch.ts` — `WebSearchQuery` (query string, maxResults, searchDepth, includeDomains, excludeDomains), `WebSearchResult` (title, url, snippet, score, publishedDate, rawContent), `WebSearchResponse` (results array, query metadata, provider used, latency).
2. Create the provider interface in `server/src/services/search/webSearchClient.ts` — `IWebSearchProvider` with `search(query: WebSearchQuery): Promise<WebSearchResponse>` and `isAvailable(): Promise<boolean>`.
3. Implement `server/src/services/search/tavilyClient.ts` — Tavily Search API integration using their REST API. Support `search_depth: "basic" | "advanced"`, domain filtering, and result scoring. Parse Tavily's response into the unified `WebSearchResult` format.
4. Implement `server/src/services/search/braveClient.ts` — Brave Search API integration as fallback. Map Brave's response format to the unified `WebSearchResult` schema. Handle Brave's rate limiting (1 req/sec on free tier).
5. Create `server/src/services/search/index.ts` — factory/manager that tries Tavily first, falls back to Brave on failure, logs provider selection. Expose a single `webSearch(query)` function.
6. Add `POST /api/search/web` route in `server/src/routes/search.ts` — accepts `WebSearchQuery` body, validates with Zod, calls the search manager, returns `WebSearchResponse`. Include auth middleware.
7. Add `TAVILY_API_KEY` and `BRAVE_SEARCH_API_KEY` to `.env.example`.
8. Write unit tests for schema validation, provider fallback logic, and response mapping.

## Files to Create/Modify
- `shared/src/schemas/webSearch.ts` — WebSearchQuery, WebSearchResult, WebSearchResponse Zod schemas
- `server/src/services/search/webSearchClient.ts` — IWebSearchProvider interface and types
- `server/src/services/search/tavilyClient.ts` — Tavily API client implementation
- `server/src/services/search/braveClient.ts` — Brave Search API client implementation
- `server/src/services/search/index.ts` — Search manager with fallback logic
- `server/src/routes/search.ts` — POST /api/search/web endpoint
- `.env.example` — Add TAVILY_API_KEY, BRAVE_SEARCH_API_KEY
- `server/src/__tests__/services/search/webSearch.test.ts` — Unit tests

## Acceptance Criteria
- [ ] `WebSearchQuery` schema validates query string (required), maxResults (default 5, max 20), and optional domain filters
- [ ] `WebSearchResult` schema includes title, url, snippet, score (0-1), and optional publishedDate
- [ ] Tavily client sends correct API requests and maps responses to unified format
- [ ] Brave client sends correct API requests and maps responses to unified format
- [ ] Search manager tries Tavily first, falls back to Brave on error/unavailability
- [ ] Search manager logs which provider was used and latency
- [ ] `POST /api/search/web` returns 200 with valid results for a valid query
- [ ] `POST /api/search/web` returns 400 for invalid query body
- [ ] `POST /api/search/web` requires authentication
- [ ] All unit tests pass with `npm run test`
