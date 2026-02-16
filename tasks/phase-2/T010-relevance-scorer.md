# T010 — Relevance Scorer with Cross-Encoder Reranking

| Field | Value |
|-------|-------|
| Phase | 2 — Context Intelligence Engine |
| Agent Type | intelligence-agent |
| Depends On | T008 |
| Blocks | T011, T013, T015 |
| Branch | `feat/T010-relevance-scorer` |
| Status | PENDING |

## Objective
Implement a two-stage retrieval pipeline — first fetch candidate chunks from the vector store via semantic search (fast, approximate), then rerank them using a cross-encoder model for more accurate relevance scoring that considers the query-chunk pair jointly.

## Context
Bi-encoder semantic search (embed query, embed chunks, cosine similarity) is fast but imprecise — it compares independent embeddings. Cross-encoder reranking takes the query and each candidate chunk as a pair and produces a much more accurate relevance score. This two-stage approach is standard in production retrieval systems: cast a wide net with bi-encoder, then precisely rank with cross-encoder. The quality difference is dramatic for context assembly.

## Subtasks
1. Create `shared/src/schemas/scoredChunk.ts` — Zod schemas:
   - `ScoredChunk` — chunkId, documentId, content, biEncoderScore (from vector search), crossEncoderScore (from reranking), finalScore (weighted combination), metadata, rank.
   - `ScoringConfig` — candidateCount (how many to fetch from vector store, default 30), rerankCount (how many to rerank, default 30), returnCount (how many to return, default 10), biEncoderWeight (default 0.3), crossEncoderWeight (default 0.7).
2. Create `server/src/services/intelligence/crossEncoder.ts`:
   - Define `CrossEncoder` interface: `rerank(query: string, passages: string[]): Promise<{ index: number, score: number }[]>`.
   - Implement `APICrossEncoder` — calls an external reranking API (Cohere Rerank or a self-hosted cross-encoder endpoint). Configurable via environment variables.
   - Implement `LLMCrossEncoder` — fallback that uses an LLM (OpenAI/Claude) to score relevance of each passage to the query on a 0-1 scale. Slower but requires no extra service.
   - Factory function `createCrossEncoder(config)` that selects implementation.
3. Create `server/src/services/intelligence/relevanceScorer.ts`:
   - `scoreChunks(query, projectId, config?)`:
     a. Call semantic search endpoint to get `candidateCount` chunks with bi-encoder scores.
     b. Pass the top `rerankCount` chunks through the cross-encoder for reranking.
     c. Compute final scores: `finalScore = biEncoderWeight * biEncoderScore + crossEncoderWeight * crossEncoderScore`.
     d. Sort by finalScore descending.
     e. Return top `returnCount` ScoredChunks.
   - `scoreChunksFromMultipleSources(query, chunks[])` — Score chunks that come from different sources (docs, web results) using the same cross-encoder.
   - Handle graceful degradation: if cross-encoder is unavailable, fall back to bi-encoder scores only with a warning.
4. Add environment variables: `CROSS_ENCODER_PROVIDER` (cohere|llm), `COHERE_API_KEY`, `CROSS_ENCODER_MODEL`.
5. Write unit tests with mocked search and cross-encoder to verify scoring logic, weighting, and fallback behavior.

## Files to Create/Modify
- `server/src/services/intelligence/relevanceScorer.ts` — Two-stage retrieval and scoring orchestrator
- `server/src/services/intelligence/crossEncoder.ts` — Cross-encoder interface and implementations
- `shared/src/schemas/scoredChunk.ts` — Zod schemas for scored chunks and scoring config
- `.env.example` — Add cross-encoder configuration variables

## Acceptance Criteria
- [ ] Two-stage pipeline: bi-encoder retrieves 30 candidates, cross-encoder reranks top 30, returns top 10
- [ ] Cross-encoder reranking meaningfully changes the ordering compared to bi-encoder alone
- [ ] Final score correctly combines bi-encoder and cross-encoder scores with configurable weights
- [ ] `APICrossEncoder` calls Cohere Rerank API and returns normalized scores
- [ ] `LLMCrossEncoder` uses OpenAI/Claude to score relevance as a fallback
- [ ] Graceful degradation: if cross-encoder fails, results use bi-encoder scores only
- [ ] `ScoredChunk` includes both individual scores and the weighted final score
- [ ] `candidateCount`, `rerankCount`, `returnCount` are all independently configurable
- [ ] Multi-source scoring works for chunks from different origins (docs vs web)
- [ ] Performance: reranking 30 chunks completes within 3 seconds
