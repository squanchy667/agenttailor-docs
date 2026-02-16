# T022 — Document Upload and Processing Status UI

| Field | Value |
|-------|-------|
| Phase | 4 — Dashboard Frontend |
| Agent Type | frontend-agent |
| Depends On | T020, T007 |
| Blocks | — |
| Branch | `feat/T022-document-upload-ui` |
| Status | PENDING |

## Objective
Build the document upload interface with drag-and-drop support, upload progress indicators, and real-time processing status updates via WebSocket. Users can see each document transition through processing stages: uploading, processing, chunking, embedding, ready (or error).

## Context
Document ingestion is the core value loop of AgentTailor — users upload their docs, the system processes and embeds them, and then those docs power context assembly. The upload experience must feel fast and transparent. Real-time status updates via WebSocket keep the user informed without polling, and the step-by-step progress indicator builds trust that the system is working. This connects to the document ingestion API (T007) and the processing pipeline.

## Subtasks
1. Create `dashboard/src/lib/websocket.ts` — WebSocket client that connects to the server's WS endpoint, handles reconnection with exponential backoff, authenticates using Clerk token, and provides a typed event system (`onDocumentStatus`, `onProcessingProgress`).
2. Create `server/src/lib/websocket.ts` — WebSocket server (using `ws` library) integrated with the Express server. Authenticates connections, supports room-based subscriptions (per project/document), and exposes `broadcastDocumentStatus(documentId, status, progress)` for the processing pipeline to call.
3. Create `dashboard/src/components/documents/DocumentUploader.tsx` — drag-and-drop zone (accepts .pdf, .md, .txt, .docx, .html). Visual feedback on drag-over, file type validation, multi-file upload support. Shows upload progress bar per file. Uses `FormData` and `XMLHttpRequest` or `fetch` with progress tracking.
4. Create `dashboard/src/components/documents/DocumentList.tsx` — table/list of documents for a project. Columns: name, type, size, status, uploaded date, actions (view, re-process, delete). Sortable by name/date/status. Pagination.
5. Create `dashboard/src/components/documents/DocumentStatus.tsx` — real-time status badge that subscribes to WebSocket updates for its document. States: uploading (blue, spinner), processing (yellow, spinner), chunking (yellow, spinner), embedding (yellow, spinner), ready (green, check), error (red, x with tooltip showing error message).
6. Create `dashboard/src/components/documents/ProcessingProgress.tsx` — step indicator showing the full pipeline: Upload > Process > Chunk > Embed > Ready. Current step is highlighted and animated. Failed step shows error with retry button.
7. Create `dashboard/src/hooks/useDocuments.ts` — React Query hooks: `useDocuments(projectId)`, `useUploadDocument(projectId)`, `useDeleteDocument()`, `useReprocessDocument()`. Upload mutation handles FormData creation and progress callbacks.
8. Integrate document components into `ProjectDetailPage.tsx` (T021) documents tab.

## Files to Create/Modify
- `dashboard/src/components/documents/DocumentUploader.tsx` — Drag-and-drop upload zone
- `dashboard/src/components/documents/DocumentList.tsx` — Document list with status
- `dashboard/src/components/documents/DocumentStatus.tsx` — Real-time status badge
- `dashboard/src/components/documents/ProcessingProgress.tsx` — Pipeline step indicator
- `dashboard/src/hooks/useDocuments.ts` — React Query hooks for document operations
- `dashboard/src/lib/websocket.ts` — WebSocket client with reconnection
- `server/src/lib/websocket.ts` — WebSocket server with room-based subscriptions
- `dashboard/src/pages/ProjectDetailPage.tsx` — Modify: integrate document components

## Acceptance Criteria
- [ ] Drag-and-drop zone accepts files and shows visual feedback on drag-over
- [ ] File type validation rejects unsupported formats with clear error message
- [ ] Upload progress bar shows real-time upload percentage per file
- [ ] Multi-file upload works — each file gets its own progress indicator
- [ ] Document list displays all documents with name, type, size, status, and date
- [ ] Document list supports sorting by name, date, and status
- [ ] DocumentStatus badge updates in real-time via WebSocket as processing progresses
- [ ] ProcessingProgress shows the current pipeline step with visual progress
- [ ] Error state shows error message and offers a retry/reprocess button
- [ ] WebSocket client reconnects automatically on disconnect (exponential backoff)
- [ ] WebSocket connection authenticates using Clerk token
- [ ] Delete document shows confirmation and removes from list
- [ ] Empty state shows when project has no documents (with upload CTA)
