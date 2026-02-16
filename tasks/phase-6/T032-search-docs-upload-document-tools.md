# T032 — search_docs and upload_document MCP Tools

| Field | Value |
|-------|-------|
| Phase | 6 — MCP Server |
| Agent Type | integration-agent |
| Depends On | T031, T008, T007 |
| Blocks | — |
| Branch | `feat/T032-search-docs-upload-document-tools` |
| Status | PENDING |

## Objective
Add two additional MCP tools to the server: `search_docs` for semantic search within project documents, and `upload_document` for uploading files to a project for processing through the ingestion pipeline.

## Context
The `tailor_context` tool handles the full pipeline, but users often need more granular operations. `search_docs` lets Claude search a project's knowledge base for specific information without running the full tailoring pipeline — useful for quick lookups and fact-checking. `upload_document` lets users add new documents to their project directly from Claude's interface, removing the need to switch to the dashboard for document management. Together these tools make AgentTailor a complete knowledge management layer accessible entirely from within Claude.

## Subtasks
1. Add schemas to `shared/src/schemas/mcp.ts`:
   - `SearchDocsInput` — query (string, required), projectId (string UUID, required), topK (number, optional, default 5), minScore (number, optional, default 0.5).
   - `SearchDocsOutput` — results array of { content, documentTitle, chunkIndex, score }.
   - `UploadDocumentInput` — projectId (string UUID, required), fileName (string, required), content (string, required — base64 or plain text), mimeType (string, optional, default "text/plain").
   - `UploadDocumentOutput` — documentId (string), fileName (string), status ("processing" | "complete"), chunkCount (number, optional).
2. Add methods to `mcp-server/src/lib/apiClient.ts`:
   - `searchDocs(input)` — POST to `/api/search/docs` with query, projectId, topK, minScore. Returns ranked results.
   - `uploadDocument(input)` — POST to `/api/projects/:id/documents` with file content and metadata. Returns document record.
3. Create `mcp-server/src/tools/searchDocs.ts` — MCP tool definition:
   - Tool name: `search_docs`.
   - Description: "Search project documents by semantic similarity. Returns the most relevant chunks with relevance scores."
   - Handler calls `apiClient.searchDocs()`, formats results as numbered list with source attribution and scores.
4. Create `mcp-server/src/tools/uploadDocument.ts` — MCP tool definition:
   - Tool name: `upload_document`.
   - Description: "Upload a document to a project for indexing. Supports text, markdown, and base64-encoded files."
   - Handler calls `apiClient.uploadDocument()`, returns confirmation with document ID and processing status.
   - Validate file size (reject > 10MB content).
5. Register both new tools in `mcp-server/src/index.ts` alongside the existing `tailor_context` tool.

## Files to Create/Modify
- `mcp-server/src/tools/searchDocs.ts` — search_docs tool implementation
- `mcp-server/src/tools/uploadDocument.ts` — upload_document tool implementation
- `mcp-server/src/lib/apiClient.ts` — Add searchDocs and uploadDocument methods (modify)
- `mcp-server/src/index.ts` — Register new tools (modify)
- `shared/src/schemas/mcp.ts` — Add SearchDocs and UploadDocument schemas (modify)

## Acceptance Criteria
- [ ] `search_docs` tool accepts query and projectId parameters and returns ranked results
- [ ] Search results include content snippets, document titles, chunk indices, and relevance scores
- [ ] `upload_document` tool accepts projectId, fileName, and content parameters
- [ ] Upload triggers the document processing pipeline (chunking and embedding)
- [ ] Upload rejects files larger than 10MB with a descriptive error message
- [ ] Both tools have properly defined JSON Schema input schemas for MCP discovery
- [ ] Both tools are listed when Claude queries available tools from the MCP server
- [ ] Invalid projectId returns a clear "project not found" error
