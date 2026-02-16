# T007 — Document Upload and Processing Pipeline

| Field | Value |
|-------|-------|
| Phase | 1 — Foundation |
| Agent Type | backend-agent |
| Depends On | T004, T005, T006 |
| Blocks | T008, T022, T032 |
| Branch | `feat/T007-document-upload-pipeline` |
| Status | PENDING |

## Objective
Build the document ingestion pipeline — file upload endpoint with validation, text extraction (PDF, DOCX, Markdown, TXT, source code), and async processing via Bull queue that extracts text, chunks it, generates embeddings, and stores vectors. Track document status throughout the pipeline.

## Context
This is the primary data ingestion path for AgentTailor. Users upload their project documents and the system must reliably process them through a multi-step pipeline (extract -> chunk -> embed -> store). Using Bull queues ensures processing is async (users don't wait), retryable (failures don't lose uploads), and observable (status tracking). Each file type needs a dedicated extractor because structure varies wildly between PDFs, DOCX, Markdown, and code.

## Subtasks
1. Create `server/src/lib/queue.ts` — Bull queue setup:
   - Connect to Redis from environment config.
   - Create `documentProcessing` queue with concurrency 3.
   - Add event listeners for completed, failed, and stalled jobs.
   - Export typed queue instance.
2. Create text extractors:
   - `server/src/services/textExtractor/index.ts` — `TextExtractor` interface: `extract(buffer: Buffer, filename: string): Promise<ExtractedText>`. Factory function that selects extractor by file extension.
   - `server/src/services/textExtractor/pdfExtractor.ts` — Extract text from PDF using `pdf-parse`. Preserve page numbers in metadata.
   - `server/src/services/textExtractor/docxExtractor.ts` — Extract text from DOCX using `mammoth`. Preserve heading structure.
   - `server/src/services/textExtractor/markdownExtractor.ts` — Pass through Markdown content. Parse frontmatter if present.
   - `server/src/services/textExtractor/codeExtractor.ts` — Handle source code files (.ts, .js, .py, .go, etc.). Add language detection metadata. Preserve function/class boundaries.
3. Create `server/src/services/documentProcessor.ts` — Orchestrates the full pipeline:
   - `processDocument(documentId)` — Fetch document record, extract text, chunk it, generate embeddings, store in vector DB, update document status and chunk count.
   - Handle errors at each stage — update document status to ERROR with error details in metadata.
   - Emit progress events for real-time status updates.
4. Create `server/src/workers/documentWorker.ts` — Bull worker that:
   - Listens on the `documentProcessing` queue.
   - Calls `documentProcessor.processDocument(job.data.documentId)`.
   - Reports progress percentage.
   - Handles retries (max 3 attempts with backoff).
5. Create `server/src/routes/documents.ts` — Express router:
   - `POST /api/projects/:projectId/documents` — Multipart file upload (multer). Validate file type (PDF, DOCX, MD, TXT, common code extensions) and size (max 50MB). Create Document record with status PROCESSING, enqueue processing job, return 202 Accepted.
   - `GET /api/projects/:projectId/documents` — List documents for project with status and chunk counts.
   - `GET /api/projects/:projectId/documents/:docId` — Get single document with processing status and metadata.
   - `DELETE /api/projects/:projectId/documents/:docId` — Delete document, its chunks from DB, and embeddings from vector store.
6. Mount documents router in `server/src/routes/index.ts`.
7. Add file storage — save uploaded files to `server/uploads/` (local dev) with configurable path. Store file path in document record.

## Files to Create/Modify
- `server/src/routes/documents.ts` — Document upload and management endpoints
- `server/src/services/documentProcessor.ts` — Full processing pipeline orchestrator
- `server/src/services/textExtractor/index.ts` — Extractor interface and factory
- `server/src/services/textExtractor/pdfExtractor.ts` — PDF text extraction
- `server/src/services/textExtractor/docxExtractor.ts` — DOCX text extraction
- `server/src/services/textExtractor/markdownExtractor.ts` — Markdown text passthrough
- `server/src/services/textExtractor/codeExtractor.ts` — Source code extraction
- `server/src/lib/queue.ts` — Bull queue configuration and setup
- `server/src/workers/documentWorker.ts` — Async document processing worker
- `server/src/routes/index.ts` — Mount documents router

## Acceptance Criteria
- [ ] `POST /api/projects/:id/documents` accepts file upload and returns 202 with document ID
- [ ] File type validation rejects unsupported formats with 400
- [ ] File size validation rejects files over 50MB with 413
- [ ] Document status transitions: PROCESSING -> READY (success) or PROCESSING -> ERROR (failure)
- [ ] PDF extractor correctly extracts text and preserves page numbers
- [ ] DOCX extractor preserves heading structure in output
- [ ] Markdown extractor handles frontmatter and passes through content
- [ ] Code extractor detects language and preserves structure
- [ ] After successful processing, chunks exist in PostgreSQL and embeddings exist in vector store
- [ ] Document chunkCount field reflects actual number of chunks created
- [ ] Failed processing sets document status to ERROR with error details
- [ ] Bull worker retries failed jobs up to 3 times with backoff
- [ ] `DELETE /api/projects/:id/documents/:docId` removes DB records and vector embeddings
- [ ] Concurrent uploads (3 simultaneous) process without conflicts
