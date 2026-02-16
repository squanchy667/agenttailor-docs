# T012 — Context Compressor

| Field | Value |
|-------|-------|
| Phase | 2 — Context Intelligence Engine |
| Agent Type | intelligence-agent |
| Depends On | T001 |
| Blocks | T013, T015 |
| Branch | `feat/T012-context-compressor` |
| Status | PENDING |

## Objective
Build the hierarchy-based context compressor that applies different compression strategies based on chunk relevance — exact quotes for high-relevance chunks, LLM-generated summaries for medium-relevance chunks, and keyword extraction for low-relevance chunks. Respects an overall token budget.

## Context
The tailored context package must fit within the target model's context window while maximizing information density. Naive approaches either truncate (losing important details) or include everything (wasting tokens on marginally relevant content). The hierarchical compressor is smarter: it allocates the most token budget to the most relevant content and progressively compresses less relevant material. This is a key differentiator — the same 4K token budget delivers dramatically more useful context.

## Subtasks
1. Create `shared/src/schemas/compressedContext.ts` — Zod schemas:
   - `CompressionLevel` enum: FULL (exact content), SUMMARY (LLM-summarized), KEYWORDS (key terms only), REFERENCE (just title/source link).
   - `CompressedChunk` — originalChunkId, compressionLevel, content (compressed), originalTokenCount, compressedTokenCount, relevanceScore.
   - `CompressionConfig` — totalTokenBudget, thresholds (relevance score boundaries for each level: e.g., >0.8 FULL, 0.5-0.8 SUMMARY, 0.3-0.5 KEYWORDS, <0.3 REFERENCE), summaryMaxTokens (per chunk, default 150), reservedTokens (for system prompt and formatting overhead).
   - `CompressedContext` — chunks (CompressedChunk array), totalTokenCount, compressionStats (how many at each level, total savings).
2. Create `server/src/services/intelligence/summarizer.ts`:
   - `summarizeChunk(content, maxTokens)` — Uses OpenAI/Claude to generate a concise summary of the chunk content, preserving key facts, numbers, and technical terms. Stays within maxTokens.
   - `summarizeBatch(chunks[], maxTokensEach)` — Batch summarize multiple chunks in a single API call for efficiency.
   - `extractKeywords(content, maxKeywords)` — Extract the N most important keywords/phrases from the content. Uses TF-IDF-like heuristics first, LLM as fallback.
3. Create `server/src/services/intelligence/contextCompressor.ts`:
   - `compressContext(scoredChunks, config)`:
     a. Sort chunks by relevance score descending.
     b. Assign compression levels based on score thresholds from config.
     c. Calculate token budget allocation: FULL chunks get their full token count, SUMMARY chunks get summaryMaxTokens, KEYWORDS chunks get ~30 tokens, REFERENCE chunks get ~10 tokens.
     d. If total exceeds budget, progressively demote chunks from lowest relevance upward (FULL -> SUMMARY -> KEYWORDS -> REFERENCE -> excluded).
     e. Execute compression: leave FULL chunks as-is, run summarizer on SUMMARY chunks, extract keywords for KEYWORDS chunks, format source reference for REFERENCE chunks.
     f. Return CompressedContext with stats.
   - `estimateCompressedSize(scoredChunks, config)` — Quick estimate without actually running summarization. Useful for preview.
4. Write unit tests covering:
   - Budget allocation across compression levels.
   - Progressive demotion when over budget.
   - Compression statistics accuracy.
   - Edge cases: all chunks high relevance, all low relevance, single chunk.

## Files to Create/Modify
- `server/src/services/intelligence/contextCompressor.ts` — Hierarchical compression orchestrator with budget management
- `server/src/services/intelligence/summarizer.ts` — LLM-based summarization and keyword extraction
- `shared/src/schemas/compressedContext.ts` — Zod schemas for compression levels, config, and output

## Acceptance Criteria
- [ ] High-relevance chunks (score > 0.8) are included at FULL compression (exact content)
- [ ] Medium-relevance chunks (0.5-0.8) are summarized to ~150 tokens each
- [ ] Low-relevance chunks (0.3-0.5) are reduced to keywords only
- [ ] Below-threshold chunks (< 0.3) are reduced to source references
- [ ] Total output respects the token budget — never exceeds it
- [ ] When budget is tight, lower-relevance chunks are progressively demoted
- [ ] Summarizer produces concise summaries preserving key facts and technical terms
- [ ] Keyword extraction returns meaningful terms, not stop words
- [ ] `estimateCompressedSize` provides a reasonable estimate without LLM calls
- [ ] Compression stats report counts per level and total token savings
- [ ] Works correctly with a single chunk (no unnecessary compression)
- [ ] Works correctly when all chunks are high relevance (fills budget with full content)
