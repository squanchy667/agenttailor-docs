# T033 — MCP Resource Providers for Documents and Sessions

| Field | Value |
|-------|-------|
| Phase | 6 — MCP Server |
| Agent Type | integration-agent |
| Depends On | T031 |
| Blocks | — |
| Branch | `feat/T033-mcp-resource-providers` |
| Status | PENDING |

## Objective
Implement MCP resource providers that expose project documents and tailoring sessions as browseable resources in Claude's interface, allowing users to inspect their knowledge base and review past tailoring results without leaving the conversation.

## Context
MCP resources are a read-only complement to tools. While tools perform actions, resources let Claude browse and read structured data. By exposing documents and sessions as resources, users can ask Claude to "look at my project docs" or "review my last tailoring session" and get direct access to the content. This makes the MCP integration feel like a first-class knowledge management system rather than just an API wrapper. Resources also enable Claude to proactively reference relevant documents when answering questions.

## Subtasks
1. Add methods to `mcp-server/src/lib/apiClient.ts`:
   - `listDocuments(projectId)` — GET `/api/projects/:id/documents`. Returns array of { id, fileName, mimeType, chunkCount, createdAt }.
   - `getDocumentContent(projectId, documentId)` — GET `/api/projects/:id/documents/:docId`. Returns full document content.
   - `listSessions(projectId, limit?)` — GET `/api/projects/:id/sessions?limit=N`. Returns array of { id, task, createdAt, qualityScore }.
   - `getSessionDetail(sessionId)` — GET `/api/sessions/:id`. Returns full session with assembled context, sources, and quality metadata.
   - `listProjects()` — GET `/api/projects`. Returns array of { id, name, description, documentCount }.
2. Create `mcp-server/src/resources/documents.ts` — Document resource provider:
   - Resource template: `agenttailor://projects/{projectId}/documents/{documentId}` for individual documents.
   - List handler: returns all documents in a project as resource entries with name (fileName), description (mimeType + chunk count), and URI.
   - Read handler: fetches and returns document content as text/plain.
   - Also register a top-level `agenttailor://projects` resource that lists all projects.
3. Create `mcp-server/src/resources/sessions.ts` — Session resource provider:
   - Resource template: `agenttailor://projects/{projectId}/sessions/{sessionId}` for individual sessions.
   - List handler: returns recent sessions (last 20) with name (truncated task), description (quality score + date), and URI.
   - Read handler: fetches session detail, formats as structured text with task, assembled context, source list, and quality score breakdown.
4. Register both resource providers in `mcp-server/src/index.ts`:
   - Register resource templates for documents and sessions.
   - Register list and read handlers.
5. Add error handling: return appropriate MCP errors for missing projects, documents, and sessions.

## Files to Create/Modify
- `mcp-server/src/resources/documents.ts` — Document resource provider with list and read
- `mcp-server/src/resources/sessions.ts` — Session resource provider with list and read
- `mcp-server/src/lib/apiClient.ts` — Add document, session, and project list methods (modify)
- `mcp-server/src/index.ts` — Register resource providers (modify)

## Acceptance Criteria
- [ ] Documents appear as browseable MCP resources under `agenttailor://projects/{id}/documents`
- [ ] Reading a document resource returns its full content
- [ ] Sessions appear as browseable MCP resources under `agenttailor://projects/{id}/sessions`
- [ ] Reading a session resource returns assembled context, sources, and quality score
- [ ] Top-level `agenttailor://projects` resource lists all user projects
- [ ] Resources update dynamically when new documents are uploaded or sessions are created
- [ ] Missing resources return proper MCP error responses (not crashes)
- [ ] Resource URIs follow the `agenttailor://` scheme consistently
