# T006 — Embedding Generation and Vector Storage

| Field | Value |
|-------|-------|
| Phase | 1 — Foundation |
| Agent Type | pipeline-agent |
| Depends On | T002 |
| Blocks | T007, T008 |
| Branch | `feat/T006-embedding-vector-storage` |
| Status | PENDING |

## Objective
Implement embedding generation using OpenAI text-embedding-ada-002 (with an interface for swappable providers) and vector storage using ChromaDB for local development with a Pinecone adapter for production.

## Context
Embeddings convert text chunks into numerical vectors that enable semantic search — the core retrieval mechanism for AgentTailor. The vector store is where these embeddings live and get queried. Using an interface/adapter pattern means we can develop locally with ChromaDB (Docker, no API key needed) and deploy with Pinecone (managed, scalable) without changing application code.

## Subtasks
1. Create `server/src/lib/openai.ts` — OpenAI client singleton configured from environment variables. Includes rate limiting awareness and retry logic.
2. Create `shared/src/schemas/embedding.ts` — Zod schemas:
   - `EmbeddingConfig` — model (string, default "text-embedding-ada-002"), dimensions (number, default 1536), batchSize (number, default 100).
   - `EmbeddingResult` — chunkId, embedding (number array), tokenCount.
3. Create `server/src/services/embedding/embedder.ts`:
   - Define `Embedder` interface: `embedText(text: string): Promise<number[]>`, `embedBatch(texts: string[]): Promise<number[][]>`.
   - Implement `OpenAIEmbedder` class implementing the interface. Handles batching (max 100 texts per API call), retries with exponential backoff, and token counting.
   - Factory function `createEmbedder(config)` that returns the appropriate implementation.
4. Create `server/src/services/vectorStore/index.ts`:
   - Define `VectorStore` interface with methods:
     - `upsert(collectionName, items: { id, embedding, metadata }[]): Promise<void>`
     - `query(collectionName, embedding, topK, filter?): Promise<VectorSearchResult[]>`
     - `delete(collectionName, ids: string[]): Promise<void>`
     - `deleteCollection(collectionName): Promise<void>`
   - `VectorSearchResult` type: id, score, metadata.
5. Create `server/src/services/vectorStore/chromaStore.ts`:
   - Implement `VectorStore` interface using ChromaDB client.
   - Connect to ChromaDB via HTTP (configurable URL from env).
   - Collection naming convention: `project_{projectId}`.
   - Handle collection creation if not exists.
6. Create `server/src/services/vectorStore/pineconeStore.ts`:
   - Implement `VectorStore` interface using Pinecone client.
   - Namespace convention: `project_{projectId}`.
   - Handle index initialization.
7. Create factory in `server/src/services/vectorStore/index.ts`: `createVectorStore(provider)` that returns ChromaDB or Pinecone implementation based on config/env.
8. Add environment variables to `.env.example`: `OPENAI_API_KEY`, `VECTOR_STORE_PROVIDER` (chroma|pinecone), `CHROMA_URL`, `PINECONE_API_KEY`, `PINECONE_INDEX`.

## Files to Create/Modify
- `server/src/services/embedding/embedder.ts` — Embedder interface and OpenAI implementation
- `server/src/services/vectorStore/index.ts` — VectorStore interface, types, and factory
- `server/src/services/vectorStore/chromaStore.ts` — ChromaDB adapter
- `server/src/services/vectorStore/pineconeStore.ts` — Pinecone adapter
- `server/src/lib/openai.ts` — OpenAI client singleton with retry logic
- `shared/src/schemas/embedding.ts` — Zod schemas for embedding config and results
- `.env.example` — Add vector store and OpenAI variables

## Acceptance Criteria
- [ ] `OpenAIEmbedder.embedText()` returns a 1536-dimension vector for a text input
- [ ] `OpenAIEmbedder.embedBatch()` handles batches larger than 100 by splitting into sub-batches
- [ ] Retry logic handles OpenAI rate limits with exponential backoff
- [ ] ChromaDB store creates collections and upserts embeddings successfully
- [ ] ChromaDB store returns top-K results ranked by similarity score
- [ ] Pinecone store implements the same interface and passes the same integration tests
- [ ] `createVectorStore("chroma")` returns ChromaStore, `createVectorStore("pinecone")` returns PineconeStore
- [ ] Vector store `delete` and `deleteCollection` clean up data correctly
- [ ] Metadata filtering works in vector queries (e.g., filter by document ID)
- [ ] OpenAI client singleton prevents multiple instances
