# T013 — Source Synthesizer and Priority Ranker

| Field | Value |
|-------|-------|
| Phase | 2 — Context Intelligence Engine |
| Agent Type | intelligence-agent |
| Depends On | T010, T012 |
| Blocks | T015 |
| Branch | `feat/T013-source-synthesizer` |
| Status | PENDING |

## Objective
Merge information from multiple sources (project documents, web search results, API responses) into a coherent, deduplicated context package. Resolve contradictions, note disagreements between sources, and rank final context by direct relevance, recency, source authority, and specificity.

## Context
AgentTailor assembles context from diverse sources — user-uploaded docs may have outdated API references, web search may find current docs but with less project-specific context, and different documents may contradict each other. The Source Synthesizer is the "editor" that merges these sources intelligently: deduplicating overlapping information, flagging contradictions, and producing a coherent narrative rather than a jumbled pile of chunks. The Priority Ranker ensures the most valuable information appears first in the context window.

## Subtasks
1. Create `shared/src/schemas/synthesizedContext.ts` — Zod schemas:
   - `SourceType` enum: PROJECT_DOC, WEB_SEARCH, API_RESPONSE, USER_INPUT.
   - `ContextSource` — sourceType, sourceId, title, url (optional), timestamp (optional), authorityScore (0-1).
   - `SynthesizedBlock` — content, sources (array of ContextSource — tracks provenance), priority (number), section (string — category label like "Core Implementation", "Background", "Examples"), contradictions (optional array of { claim, sources, alternative, alternativeSources }).
   - `SynthesizedContext` — blocks (array of SynthesizedBlock), totalTokenCount, sourceCount, contradictionCount, sections (ordered list of section names).
2. Create `server/src/services/intelligence/priorityRanker.ts`:
   - `rankByPriority(items, weights?)` — Multi-factor ranking:
     a. **Direct relevance** (weight 0.4): How closely does this match the task? Uses the cross-encoder score.
     b. **Recency** (weight 0.2): Newer content ranked higher. Uses document upload date, web result fetch date, or content metadata dates.
     c. **Source authority** (weight 0.2): Project docs > official documentation > blog posts > forums. Configurable authority mapping.
     d. **Specificity** (weight 0.2): Specific, actionable content ranked over general overviews. Detected by presence of code examples, specific numbers, step-by-step instructions.
   - Weights are configurable per task type (e.g., RESEARCH tasks weight recency higher, CODING tasks weight specificity higher).
3. Create `server/src/services/intelligence/sourceSynthesizer.ts`:
   - `synthesize(compressedChunks, webResults?, taskAnalysis?)`:
     a. **Deduplication**: Detect semantically similar content from different sources using embedding similarity. Keep the higher-quality version, note the duplicate source for provenance.
     b. **Contradiction detection**: When two sources make conflicting claims about the same topic, flag both with the contradiction metadata. Use simple heuristics (same entity, different values) and optional LLM for nuanced cases.
     c. **Section grouping**: Organize content into logical sections based on the task analysis domains and content clustering. E.g., "Core Implementation Details", "Configuration Reference", "Background Context", "Related Examples".
     d. **Priority ranking**: Apply priorityRanker to order blocks within each section and sections relative to each other.
     e. **Provenance tracking**: Every block tracks which source(s) contributed to it.
     f. Return `SynthesizedContext` with organized, ranked, deduplicated content.
   - `mergeWebResults(webResults, existingContext)` — Specifically handles merging web search results into an existing context from project documents.
4. Write unit tests covering:
   - Deduplication of overlapping content from different sources.
   - Contradiction detection between conflicting sources.
   - Priority ranking with different weight configurations.
   - Section grouping coherence.

## Files to Create/Modify
- `server/src/services/intelligence/sourceSynthesizer.ts` — Multi-source merging with dedup and contradiction detection
- `server/src/services/intelligence/priorityRanker.ts` — Multi-factor priority ranking
- `shared/src/schemas/synthesizedContext.ts` — Zod schemas for synthesized context, sources, and blocks

## Acceptance Criteria
- [ ] Duplicate content from different sources is detected and merged (not repeated)
- [ ] Contradictions between sources are flagged with both claims and their sources
- [ ] Priority ranking considers relevance, recency, authority, and specificity
- [ ] Ranking weights are configurable and adjust based on task type
- [ ] Content is organized into logical sections with meaningful labels
- [ ] Each content block tracks its source provenance (which documents/URLs contributed)
- [ ] Web search results are correctly merged with project document chunks
- [ ] Source authority scoring: project docs > official docs > blogs > forums
- [ ] Specificity detection correctly identifies actionable vs. general content
- [ ] Output includes contradiction count and source count in metadata
- [ ] Sections are ordered by overall priority (most relevant section first)
- [ ] Unit tests verify dedup, contradiction detection, and ranking logic
