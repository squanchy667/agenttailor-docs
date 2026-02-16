# T015 — Tailoring Orchestration Endpoint (/api/tailor)

| Field | Value |
|-------|-------|
| Phase | 2 — Context Intelligence Engine |
| Agent Type | intelligence-agent |
| Depends On | T009, T010, T011, T012, T013, T014 |
| Blocks | T017, T023, T029, T031, T034, T036 |
| Branch | `feat/T015-tailoring-orchestration-endpoint` |
| Status | PENDING |

## Objective
Build THE CORE endpoint — `POST /api/tailor` — that orchestrates the full context assembly pipeline: analyze task, retrieve chunks, score relevance, detect gaps, compress context, synthesize sources, manage token budget, and format for the target AI platform. Also implement `POST /api/tailor/preview` (lightweight estimation) and `GET /api/tailor/sessions` (session history).

## Context
This is the endpoint that delivers AgentTailor's primary value. Every other module built so far is a component of this pipeline. The orchestrator ties them all together in the correct sequence, handles errors at each stage gracefully, and produces the final context package that users inject into their AI conversations. Getting the orchestration right — including proper error handling, partial results, and performance — is critical because this is what users interact with on every single request.

## Subtasks
1. Create `shared/src/schemas/tailor.ts` — Zod schemas:
   - `TailorRequest` — taskInput (string, 1-5000 chars), projectId (UUID), targetPlatform ("chatgpt" | "claude"), options (optional: maxTokens, includeWebSearch (boolean, default true), compressionLevel, customInstructions).
   - `TailorResponse` — sessionId (UUID), context (formatted string ready to paste), sections (array: { name, content, tokenCount, sourceCount }), metadata: { totalTokens, modelLimit, tokensUsed, chunksRetrieved, chunksIncluded, gapReport, compressionStats, processingTimeMs, qualityScore }.
   - `TailorPreviewResponse` — estimatedTokens, estimatedChunks, gapSummary, estimatedQuality, processingTimeMs (much faster than full tailor).
   - `TailorSession` — id, taskInput, targetPlatform, tokenCount, qualityScore, createdAt (for session history).
2. Create `server/src/services/tailorOrchestrator.ts` — The core pipeline:
   - `tailorContext(userId, request)`:
     a. **Analyze**: Call taskAnalyzer.analyzeTask(taskInput) to classify the task.
     b. **Budget**: Call contextWindowManager.createBudget(targetPlatform) to set token limits.
     c. **Retrieve**: Call relevanceScorer.scoreChunks(taskAnalysis.suggestedSearchQueries, projectId) to get scored chunks.
     d. **Detect gaps**: Call gapDetector.detectGaps(taskAnalysis, scoredChunks) to find missing context.
     e. **Web search** (if enabled and gaps detected): If shouldTriggerWebSearch, execute web searches for gap queries, score web results, merge into chunk pool.
     f. **Compress**: Call contextCompressor.compressContext(allScoredChunks, budgetAllocation) to fit within token budget.
     g. **Synthesize**: Call sourceSynthesizer.synthesize(compressedChunks, webResults, taskAnalysis) to merge and organize.
     h. **Format**: Call platformFormatter.format(synthesizedContext, targetPlatform) to produce the final context string.
     i. **Persist**: Save TailoringSession record to database with task input, assembled context, token counts, and quality score.
     j. **Return**: TailorResponse with context, sections, and full metadata.
   - Handle errors at each stage — if scoring fails, return partial results with a warning. If compression fails, return uncompressed but truncated context. Never return nothing.
   - Calculate `qualityScore` (0-1) based on: coverage (from gap detector), relevance (average chunk scores), compression ratio, source diversity.
3. Create `server/src/services/platformFormatter.ts`:
   - `format(synthesizedContext, platform)`:
     - **ChatGPT format**: XML-style tags (`<context>`, `<source>`, `<relevance>`), optimized for GPT-4's instruction following. System-prompt-friendly formatting.
     - **Claude format**: Markdown with clear sections, `<document>` tags per Claude's recommended format, explicit source attribution. Leverages Claude's strong Markdown parsing.
   - Both formats include: section headers, source attribution, relevance indicators, and a brief "how to use this context" preamble.
4. Create `server/src/routes/tailor.ts` — Express router:
   - `POST /api/tailor` — Full tailoring pipeline. Validate with Zod. Return TailorResponse.
   - `POST /api/tailor/preview` — Lightweight preview: runs task analysis and gap detection only (no compression/synthesis). Returns estimated token count, gap summary, and quality estimate. Much faster for UI preview.
   - `GET /api/tailor/sessions` — List user's tailoring sessions with pagination. Query params: projectId (optional filter), page, limit.
   - `GET /api/tailor/sessions/:id` — Get a single session with full assembled context.
5. Mount tailor router in `server/src/routes/index.ts`.
6. Add request timing middleware to measure and log pipeline stage durations.

## Files to Create/Modify
- `server/src/routes/tailor.ts` — Tailor API endpoints (full, preview, sessions)
- `server/src/services/tailorOrchestrator.ts` — Full pipeline orchestration with error handling
- `server/src/services/platformFormatter.ts` — Platform-specific context formatting (ChatGPT vs Claude)
- `shared/src/schemas/tailor.ts` — Zod schemas for request, response, preview, and session
- `server/src/routes/index.ts` — Mount tailor router

## Acceptance Criteria
- [ ] `POST /api/tailor` returns assembled context formatted for the target platform
- [ ] Full pipeline executes in order: analyze -> budget -> retrieve -> gaps -> (web search) -> compress -> synthesize -> format
- [ ] Response includes complete metadata: token counts, chunks retrieved/included, gap report, processing time
- [ ] ChatGPT format uses XML-style tags appropriate for GPT-4
- [ ] Claude format uses Markdown with `<document>` tags per Claude's recommended format
- [ ] Quality score (0-1) reflects coverage, relevance, compression, and source diversity
- [ ] Partial results returned when intermediate stages fail (graceful degradation)
- [ ] `POST /api/tailor/preview` returns estimates in under 2 seconds (no LLM calls)
- [ ] Session is persisted to database with full context and metadata
- [ ] `GET /api/tailor/sessions` returns paginated session history filterable by project
- [ ] Web search is triggered when gap detector identifies actionable HIGH/CRITICAL gaps
- [ ] Web search can be disabled via `options.includeWebSearch = false`
- [ ] Token usage never exceeds the target model's available context window
- [ ] Pipeline timing is logged for each stage for performance monitoring
- [ ] Invalid requests (empty task, missing project) return appropriate 400 errors
