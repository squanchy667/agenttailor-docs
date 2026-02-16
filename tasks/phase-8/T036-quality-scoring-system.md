# T036 — Quality Scoring System

| Field | Value |
|-------|-------|
| Phase | 8 — Polish & Launch |
| Agent Type | polish-agent |
| Depends On | T015 |
| Blocks | T037, T040 |
| Branch | `feat/T036-quality-scoring-system` |
| Status | PENDING |

## Objective
Implement a quality scoring system for tailored outputs that evaluates context coverage, source diversity, relevance distribution, and compression ratio, providing users with a transparent measure of how well the assembled context addresses their task.

## Context
Users need to trust the context AgentTailor assembles. A quality score provides that trust signal — a high score means the context comprehensively covers the task from diverse sources with strong relevance. A low score tells the user they may need to upload more documents or refine their task description. This feedback loop is essential for user retention: without it, users can't tell whether poor AI responses are due to bad context or bad prompting. The scores also feed into the analytics dashboard (T037) for trend tracking.

## Subtasks
1. Create `shared/src/schemas/qualityScore.ts` — Zod schemas:
   - `QualitySubScores` — coverage (0-1, how much of the task is addressed by context), diversity (0-1, variety of source documents/types used), relevance (0-1, average relevance of selected chunks), compression (0-1, how efficiently context is compressed from raw sources).
   - `QualityScore` — overall (0-100 weighted composite), subScores (QualitySubScores), suggestions (string array, improvement hints), scoredAt (ISO timestamp).
   - Weights: coverage 0.35, relevance 0.30, diversity 0.20, compression 0.15.
2. Create `server/src/services/qualityScorer.ts` — Scoring service:
   - `scoreCoverage(task, contextChunks)` — Extract key concepts from the task (split by noun phrases or keywords), check how many are represented in the assembled chunks. Return ratio of covered concepts.
   - `scoreDiversity(sources)` — Count unique documents and source types (doc, web, code). Score based on: 1 source = 0.2, 2 = 0.5, 3+ = 0.8, mixed types bonus +0.2 (capped at 1.0).
   - `scoreRelevance(chunks)` — Average the similarity scores of all chunks included in the context. Penalize if any chunk is below 0.3 relevance.
   - `scoreCompression(rawTokenCount, outputTokenCount)` — Ratio of useful output to raw input. Sweet spot is 0.2-0.5 (too high means no compression, too low means too much was dropped).
   - `generateSuggestions(subScores)` — Return actionable suggestions based on low sub-scores: low coverage -> "Try adding more documentation about [topic]", low diversity -> "Context relies heavily on a single source", low relevance -> "Consider refining your task description", low compression -> "Your documents may contain too much boilerplate".
   - `scoreContext(task, contextChunks, sources, rawTokenCount, outputTokenCount)` — Orchestrator method that runs all sub-scorers and computes the weighted overall score.
3. Modify `server/src/services/tailorOrchestrator.ts`:
   - After context assembly, call `qualityScorer.scoreContext()` with the task, assembled chunks, sources, and token counts.
   - Store the quality score alongside the session record in the database.
4. Modify `server/src/routes/tailor.ts`:
   - Include `qualityScore` in the tailoring response body.
   - Add optional `?includeScore=true` query param (default true) to allow disabling score computation for latency-sensitive callers.
5. Add Prisma schema update: add `qualityScore` JSON field to the Session model to persist scores.

## Files to Create/Modify
- `server/src/services/qualityScorer.ts` — Quality scoring algorithm with sub-scorers
- `shared/src/schemas/qualityScore.ts` — QualityScore and QualitySubScores Zod schemas
- `server/src/services/tailorOrchestrator.ts` — Add quality scoring step (modify)
- `server/src/routes/tailor.ts` — Include quality score in response (modify)
- `server/prisma/schema.prisma` — Add qualityScore JSON field to Session model (modify)

## Acceptance Criteria
- [ ] Every tailoring response includes a quality score with overall (0-100) and sub-scores
- [ ] Coverage sub-score reflects how many task concepts are addressed by the context
- [ ] Diversity sub-score increases with more unique sources and mixed source types
- [ ] Relevance sub-score averages chunk similarity scores and penalizes low-relevance chunks
- [ ] Compression sub-score measures the ratio of output tokens to raw input tokens
- [ ] Low sub-scores generate specific, actionable improvement suggestions
- [ ] Quality scores are persisted in the Session database record
- [ ] `?includeScore=false` skips score computation for latency-sensitive use cases
- [ ] Scoring adds less than 50ms of latency to the tailoring pipeline
