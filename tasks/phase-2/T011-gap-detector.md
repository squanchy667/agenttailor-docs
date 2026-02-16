# T011 — Gap Detector

| Field | Value |
|-------|-------|
| Phase | 2 — Context Intelligence Engine |
| Agent Type | intelligence-agent |
| Depends On | T010 |
| Blocks | T015, T017 |
| Branch | `feat/T011-gap-detector` |
| Status | PENDING |

## Objective
Build the module that analyzes scored context chunks and identifies when the retrieved context is insufficient for the user's task — triggering web search to fill gaps or prompting the user to upload additional documents.

## Context
Retrieved context is often incomplete. A user asks about "deploying to AWS with Terraform" but their uploaded docs only cover application code, not infrastructure. The Gap Detector identifies these knowledge holes by comparing what the task requires (from Task Analyzer) against what the retrieved chunks actually cover. This is what makes AgentTailor proactive rather than just a search engine — it knows what it does not know.

## Subtasks
1. Create `shared/src/schemas/gapAnalysis.ts` — Zod schemas:
   - `GapType` enum: MISSING_DOMAIN (entire knowledge area not covered), SHALLOW_COVERAGE (topic present but insufficient depth), OUTDATED_INFO (content may be stale), MISSING_EXAMPLES (concepts present but no practical examples), NO_CONTEXT (nothing relevant found at all).
   - `GapSeverity` enum: LOW (nice to have), MEDIUM (would improve quality), HIGH (context is meaningfully incomplete), CRITICAL (task cannot be properly addressed without this).
   - `Gap` — type, severity, description, affectedDomains, suggestedActions (array: "web_search", "upload_document", "ask_user"), suggestedQueries (for web search).
   - `GapReport` — gaps (array of Gap), overallCoverage (0-1 score), isActionable (boolean — are there gaps worth filling?), estimatedQualityWithoutFilling (0-1), estimatedQualityWithFilling (0-1).
2. Create `server/src/services/intelligence/gapDetector.ts`:
   - `detectGaps(taskAnalysis, scoredChunks, config?)`:
     a. **Coverage analysis**: For each domain identified by Task Analyzer, check if scored chunks contain relevant content. If a domain has zero or very low-scoring chunks, flag as MISSING_DOMAIN.
     b. **Depth analysis**: For covered domains, check if the top chunks provide sufficient depth — score threshold, chunk count, token coverage. Flag SHALLOW_COVERAGE if below thresholds.
     c. **Recency check**: Compare chunk metadata dates (if available) against task requirements. Flag OUTDATED_INFO for stale content.
     d. **Example detection**: For coding/implementation tasks, check if chunks contain code examples or just conceptual descriptions. Flag MISSING_EXAMPLES if no code chunks found for coding tasks.
     e. **Total failure detection**: If overall coverage score is below 0.2, flag NO_CONTEXT — the user's project docs are not relevant to this task at all.
     f. Calculate `overallCoverage` as weighted average of domain coverage scores.
     g. Determine `isActionable` — are there HIGH or CRITICAL gaps that web search could reasonably fill?
     h. Generate `suggestedQueries` for each gap suitable for web search.
   - `shouldTriggerWebSearch(gapReport)` — Returns boolean based on gap severity and actionability. Configurable threshold.
   - `generateUserPrompt(gapReport)` — Creates a human-readable message explaining what's missing and suggesting actions the user could take.
3. Write unit tests covering:
   - Full coverage scenario (no gaps).
   - Single missing domain.
   - Shallow coverage detection.
   - No context found at all.
   - Web search trigger threshold.

## Files to Create/Modify
- `server/src/services/intelligence/gapDetector.ts` — Gap detection logic with coverage, depth, and recency analysis
- `shared/src/schemas/gapAnalysis.ts` — Zod schemas for GapType, GapSeverity, Gap, GapReport

## Acceptance Criteria
- [ ] Detects MISSING_DOMAIN when task requires a domain with no matching chunks
- [ ] Detects SHALLOW_COVERAGE when domain has few low-scoring chunks
- [ ] Detects MISSING_EXAMPLES for coding tasks without code-containing chunks
- [ ] NO_CONTEXT detection triggers when overall coverage is below 0.2
- [ ] Each gap includes actionable suggested queries for web search
- [ ] `overallCoverage` score accurately reflects how well chunks cover the task requirements
- [ ] `shouldTriggerWebSearch` returns true for HIGH/CRITICAL gaps that web search can address
- [ ] `generateUserPrompt` produces a clear, human-readable explanation of gaps
- [ ] Gap severity is correctly prioritized (CRITICAL > HIGH > MEDIUM > LOW)
- [ ] Full coverage scenario (all domains well-covered) returns empty gaps array with high coverage score
- [ ] Unit tests cover at least 5 distinct gap scenarios
