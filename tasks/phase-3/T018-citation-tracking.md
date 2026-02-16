# T018 — Citation Tracking and Source Attribution

| Field | Value |
|-------|-------|
| Phase | 3 — Web Search Integration |
| Agent Type | search-agent |
| Depends On | T017 |
| Blocks | — |
| Branch | `feat/T018-citation-tracking` |
| Status | PENDING |

## Objective
Track the provenance of every context chunk — whether it came from a user's document (with page number, heading, section) or from a web search (with URL, search query, fetch timestamp). Attach citation metadata to the tailored output so the AI agent can cite its sources when answering.

## Context
Trust is critical for AI-assisted work. When AgentTailor injects context into ChatGPT or Claude, the user and the AI need to know where each piece of information came from. Citation tracking transforms AgentTailor from a "magic context injector" into a transparent, auditable knowledge tool. This also enables the dashboard (Phase 4) to show source attribution in the session viewer.

## Subtasks
1. Define `Citation` Zod schema in `shared/src/schemas/citation.ts` — fields: id, type ("document" | "web"), sourceTitle, sourceUrl (for web), documentId (for docs), page (optional), heading (optional), section (optional), chunkIndex, relevanceScore, searchQuery (for web-sourced), fetchedAt (for web-sourced).
2. Define `CitedContext` schema — wraps the assembled context string with an array of `Citation` objects, each referencing a range within the context text (startOffset, endOffset).
3. Create `server/src/services/search/citationTracker.ts` — service that: (a) collects chunk metadata during context assembly, (b) maps each chunk to a Citation, (c) computes text offsets after chunks are merged into the final context string, (d) generates a citations summary section that can be appended to the tailored output.
4. Modify `server/src/services/tailorOrchestrator.ts` — integrate CitationTracker into the assembly pipeline. After context is assembled, pass chunks through CitationTracker to generate the CitedContext output. Append a "Sources" section to the tailored output with numbered references.
5. Format citations for AI consumption — generate a structured sources block at the end of the injected context: `[1] Document: "API Guide" — Section 2.3, p.12`, `[2] Web: "OAuth2 Best Practices" — https://example.com/oauth2`.
6. Store citation data alongside session records in the database so the dashboard can display them later.
7. Write tests for citation generation, offset calculation, and formatted output.

## Files to Create/Modify
- `shared/src/schemas/citation.ts` — Citation, CitedContext Zod schemas
- `server/src/services/search/citationTracker.ts` — Citation tracking and formatting service
- `server/src/services/tailorOrchestrator.ts` — Modify: integrate citation tracking into assembly output
- `server/src/services/search/citationFormatter.ts` — Format citations for AI-readable output
- `server/src/__tests__/services/search/citationTracker.test.ts` — Unit tests

## Acceptance Criteria
- [ ] `Citation` schema distinguishes between document-sourced and web-sourced chunks
- [ ] Document citations include documentId, page, heading, and section when available
- [ ] Web citations include sourceUrl, searchQuery, and fetchedAt
- [ ] CitationTracker correctly maps assembled chunks to Citation objects
- [ ] Text offsets (startOffset, endOffset) accurately reference positions in the assembled context
- [ ] Tailored output includes a numbered "Sources" section at the end
- [ ] Citation format is clear and parseable by AI models (e.g., `[1] Document: "Title" — Section, Page`)
- [ ] Citation data is persisted with the session record for dashboard retrieval
- [ ] Citations are deduplicated (same source referenced by multiple chunks produces one citation)
- [ ] All tests pass with `npm run test`
