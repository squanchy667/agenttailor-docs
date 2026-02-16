# Data Flow

## Document Ingestion Flow

```
User uploads file
       │
       ▼
POST /api/projects/:id/documents
       │
       ▼
File validation (type, size ≤ 50MB)
       │
       ▼
Store metadata in PostgreSQL (status: "processing")
       │
       ▼
Enqueue Bull job: document-process
       │
       ▼
┌─── Document Worker ──────────────────────┐
│                                          │
│  1. Text Extraction                      │
│     PDF → pdf-parse                      │
│     DOCX → mammoth                       │
│     MD/TXT → raw read                    │
│     Code → language-aware parsing        │
│                                          │
│  2. Semantic Chunking                    │
│     Split by headings/paragraphs         │
│     Configurable overlap (default 10%)   │
│     Preserve context boundaries          │
│                                          │
│  3. Embedding Generation                 │
│     OpenAI ada-002 per chunk             │
│     Batch API calls                      │
│                                          │
│  4. Vector Storage                       │
│     Store embeddings in ChromaDB/Pinecone│
│     Metadata: doc_id, position, headings │
│                                          │
│  5. Update status → "ready"              │
│     Push WebSocket notification          │
└──────────────────────────────────────────┘
```

## Tailoring Flow (THE CORE)

```
User types a message (in ChatGPT/Claude or via API)
       │
       ▼
POST /api/tailor
  { projectId, taskInput, targetPlatform: "gpt"|"claude", options }
       │
       ▼
┌─── Tailoring Orchestrator ───────────────────────┐
│                                                   │
│  1. Task Analyzer                                │
│     Input: taskInput                             │
│     Output: { taskType, domains, complexity }    │
│                                                   │
│  2. Vector Search + Relevance Scorer             │
│     Query: embedding of taskInput                │
│     Retrieve: top-K candidate chunks             │
│     Rerank: cross-encoder scoring                │
│     Output: scored chunks sorted by relevance    │
│                                                   │
│  3. Gap Detector                                 │
│     Analyze: scored chunks vs task requirements  │
│     If gaps found:                               │
│       → Trigger web search (Tavily/Brave)        │
│       → Process web results into temp chunks     │
│       → Re-score with web results included       │
│                                                   │
│  4. Context Compressor                           │
│     High relevance → exact quotes                │
│     Medium relevance → LLM summaries             │
│     Low relevance → keywords only                │
│     Respect token budget                         │
│                                                   │
│  5. Source Synthesizer + Priority Ranker          │
│     Merge: doc chunks + web results              │
│     Resolve contradictions                       │
│     Rank: relevance > recency > authority        │
│                                                   │
│  6. Context Window Manager                       │
│     Check: total tokens vs model limit           │
│     Allocate: system prompt, context, tools      │
│     Trim: lowest-priority chunks if over budget  │
│                                                   │
│  7. Platform Formatter                           │
│     GPT: structured system message               │
│     Claude: XML-tagged context blocks            │
│                                                   │
│  8. Quality Scorer                               │
│     Score: coverage, diversity, relevance        │
│                                                   │
└───────────────────────────────────────────────────┘
       │
       ▼
Response: { tailoredContext, sources, tokenCount, qualityScore, suggestions }
```

## Browser Extension Flow

```
User on chatgpt.com or claude.ai
       │
       ▼
Content script detects chat input focus
       │
       ▼
User starts typing message
       │
       ▼
Extension captures message intent
       │
       ▼
Side panel shows "Tailoring..." status
       │
       ▼
Background service worker calls POST /api/tailor
       │
       ▼
Side panel shows assembled context preview
  - Source list with relevance scores
  - Editable context text
  - "Approve" / "Edit" / "Skip" buttons
       │
       ▼
User approves → Content script injects context into chat input
       │
       ▼
Message sent with optimal context prepended
```

## State Management

- **Server state** — PostgreSQL for persistent data, Redis for ephemeral cache
- **Vector state** — ChromaDB/Pinecone for embeddings and similarity search
- **Queue state** — Bull/Redis for job tracking and processing status
- **Dashboard state** — React Query for server data, React state for UI
- **Extension state** — Chrome storage API for settings, runtime messages for communication

## Error Handling

- Document processing errors: mark document status as "error", push WebSocket notification
- Tailoring errors: return partial results with degraded quality score, log for debugging
- Vector store unavailable: fallback to keyword search in PostgreSQL
- Web search failure: continue with doc-only context, note gap in response
- Rate limit exceeded: return 429 with retry-after header
