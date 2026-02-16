# T005 — Semantic Chunking Engine

| Field | Value |
|-------|-------|
| Phase | 1 — Foundation |
| Agent Type | pipeline-agent |
| Depends On | T001 |
| Blocks | T007 |
| Branch | `feat/T005-semantic-chunking-engine` |
| Status | PENDING |

## Objective
Build the text chunking module that splits extracted document text into semantic chunks — heading-aware, with configurable overlap, preserving paragraph boundaries. This is NOT fixed-size chunking; it uses document structure to create meaningful retrieval units.

## Context
Chunking quality directly determines retrieval quality. Bad chunks (splitting mid-sentence, ignoring headings, fixed 500-token windows) produce irrelevant search results and incoherent context. The semantic chunker understands document structure — it splits on headings, respects paragraph boundaries, and includes overlap for continuity. This is a core differentiator for AgentTailor.

## Subtasks
1. Create `shared/src/schemas/chunk.ts` — Zod schemas and types:
   - `ChunkMetadata` — headings (string array, breadcrumb trail), pageNum (optional), section (optional string), language (optional), codeLanguage (optional for code chunks).
   - `ChunkConfig` — maxTokens (default 512), overlapTokens (default 50), minTokens (default 50), strategy ("semantic" | "heading" | "code").
   - `ProcessedChunk` — content, position, tokenCount, metadata.
2. Create `server/src/services/chunking/strategies/semanticChunker.ts`:
   - Split text on double newlines (paragraph boundaries).
   - Merge short consecutive paragraphs into a single chunk until maxTokens is reached.
   - Never split mid-sentence — if a paragraph exceeds maxTokens, split at sentence boundaries.
   - Add overlap from the end of the previous chunk to the start of the next.
   - Track heading context: when a heading is encountered, it becomes part of the metadata breadcrumb for all subsequent chunks until the next heading of equal or higher level.
3. Create `server/src/services/chunking/strategies/headingChunker.ts`:
   - Split exclusively on Markdown headings (# through ####).
   - Each heading and its content until the next heading becomes one chunk.
   - If a section exceeds maxTokens, fall back to semanticChunker for that section.
   - Preserve heading hierarchy in metadata.
4. Create `server/src/services/chunking/chunkEngine.ts`:
   - Main entry point: `chunkDocument(text, config)` that selects strategy based on config and document characteristics.
   - Auto-detect strategy if not specified: use headingChunker for Markdown/DOCX with headings, semanticChunker for plain text, code-aware splitting for source code.
   - Return array of `ProcessedChunk` with position indices.
5. Create `server/src/services/chunking/index.ts` — Barrel export for the chunking module.
6. Write unit tests in `server/src/__tests__/chunking/` covering:
   - Paragraph-boundary splitting.
   - Heading detection and metadata breadcrumbs.
   - Overlap generation.
   - Minimum chunk size enforcement.
   - Large paragraph sentence-level fallback.

## Files to Create/Modify
- `server/src/services/chunking/chunkEngine.ts` — Main chunking entry point with strategy selection
- `server/src/services/chunking/strategies/semanticChunker.ts` — Paragraph-aware semantic chunking
- `server/src/services/chunking/strategies/headingChunker.ts` — Markdown heading-based chunking
- `server/src/services/chunking/index.ts` — Barrel export
- `shared/src/schemas/chunk.ts` — Zod schemas for chunks, metadata, and config

## Acceptance Criteria
- [ ] Semantic chunker splits on paragraph boundaries, never mid-sentence
- [ ] Chunks respect maxTokens limit (default 512)
- [ ] Overlap of configurable token count is added between consecutive chunks
- [ ] Short paragraphs are merged together rather than becoming tiny chunks
- [ ] Heading chunker splits on Markdown headings and preserves hierarchy in metadata
- [ ] Heading breadcrumb trail is correct (e.g., `["Architecture", "Backend", "Database"]`)
- [ ] Auto-detection selects heading strategy for documents with Markdown headings
- [ ] Chunks below minTokens threshold are merged with neighbors
- [ ] Each chunk includes position index, token count, and metadata
- [ ] Unit tests pass for all chunking strategies and edge cases
