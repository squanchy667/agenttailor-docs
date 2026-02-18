# System Overview

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                      DELIVERY LAYER                          │
│                                                              │
│  ┌─────────────┐ ┌──────────┐ ┌───────────┐ ┌───────────┐  │
│  │   Browser    │ │   MCP    │ │  Custom   │ │   REST    │  │
│  │  Extension   │ │  Server  │ │   GPT     │ │   API     │  │
│  │ (Chrome MV3) │ │ (Claude) │ │ (Actions) │ │ (Public)  │  │
│  └──────┬───────┘ └────┬─────┘ └─────┬─────┘ └─────┬─────┘  │
└─────────┼──────────────┼─────────────┼──────────────┼────────┘
          │              │             │              │
          └──────────────┼─────────────┼──────────────┘
                         │             │
┌────────────────────────▼─────────────▼───────────────────────┐
│                 EXPRESS API SERVER                            │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐    │
│  │              /api/tailor (THE CORE ENDPOINT)          │    │
│  │                                                      │    │
│  │  Task Analyzer ──► Relevance Scorer ──► Gap Detector │    │
│  │       │                  │                    │      │    │
│  │       ▼                  ▼                    ▼      │    │
│  │  Context Compressor ◄── Source Synthesizer           │    │
│  │       │                  │                           │    │
│  │       ▼                  ▼                           │    │
│  │  Priority Ranker ──► Context Window Manager          │    │
│  │       │                                              │    │
│  │       ▼                                              │    │
│  │  Platform Formatter (GPT format / Claude format)     │    │
│  │       │                                              │    │
│  │       ▼                                              │    │
│  │  Quality Scorer (coverage/diversity/relevance/comp.) │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐    │
│  │              MIDDLEWARE                               │    │
│  │  Rate Limiter (Redis INCR+EXPIRE, fail-open)        │    │
│  │  Plan Enforcer (tier limits, project/doc caps)      │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐   │
│  │  /api/   │ │  /api/   │ │  /api/   │ │    /api/     │   │
│  │ projects │ │   docs   │ │  search  │ │   auth       │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────┘   │
│                                                              │
│  ┌──────────┐ ┌──────────┐                                  │
│  │  /api/   │ │  /gpt/   │                                  │
│  │analytics │ │ actions  │                                  │
│  └──────────┘ └──────────┘                                  │
└──────────────────────┬───────────────────────────────────────┘
                       │
┌──────────────────────▼───────────────────────────────────────┐
│                    DATA LAYER                                │
│                                                              │
│  ┌────────────┐ ┌──────────────────┐ ┌───────┐ ┌─────────┐ │
│  │ PostgreSQL │ │ ChromaDB/Pinecone│ │ Redis │ │  Bull   │ │
│  │  (Prisma)  │ │  (Vectors)       │ │(Cache)│ │ (Queue) │ │
│  │            │ │                  │ │       │ │         │ │
│  │ Users      │ │ Embeddings       │ │ Rate  │ │ Doc     │ │
│  │ Projects   │ │ Similarity       │ │ Limit │ │ Process │ │
│  │ Documents  │ │ Search           │ │ Cache │ │ Jobs    │ │
│  │ Sessions   │ │                  │ │       │ │         │ │
│  └────────────┘ └──────────────────┘ └───────┘ └─────────┘ │
└──────────────────────────────────────────────────────────────┘
```

## Components

### Context Intelligence Engine (Layer 2)
- **Task Analyzer** — NLP module parsing user input to identify knowledge domains, task type (coding/writing/analysis/research), and complexity level
- **Relevance Scorer** — Retrieves candidate chunks via vector similarity, then reranks using cross-encoder for higher accuracy
- **Gap Detector** — Analyzes scored context, identifies insufficient coverage, triggers web search or user prompts
- **Context Compressor** — Hierarchical compression: exact quotes (high-relevance), summaries (medium), keywords (low). Uses LLM for summarization
- **Source Synthesizer** — Merges doc chunks + web results into coherent context, resolves contradictions, notes disagreements
- **Priority Ranker** — Final ranking by (1) relevance, (2) recency, (3) source authority, (4) specificity
- **Context Window Manager** — Token budget tracking per model (128K GPT-4, 200K Claude), allocates across sections
- **Platform Formatter** — Formats output for GPT (structured system messages) vs Claude (XML-tagged context blocks)
- **Quality Scorer** — Weighted multi-dimensional scoring: coverage (0.35), relevance (0.30), diversity (0.20), compression (0.15). Produces 0-100 composite score with actionable suggestions

### Operations Layer (Layer 3)
- **Rate Limiter** — Redis INCR+EXPIRE daily counters with fail-open. X-RateLimit headers on all responses. Configurable per plan tier (FREE 50/day, PRO 500/day, TEAM 2000/day)
- **Plan Enforcer** — Structural limit enforcement: max projects, max docs per project, max file size. Tier cached in Redis with 5-min TTL
- **Analytics Service** — Usage aggregation: sessions over time, quality trends, project stats, plan usage, summary statistics. Powers the analytics dashboard

### Document Processing Pipeline (Layer 1)
- **Text Extractor** — Handles PDF, DOCX, Markdown, plain text, and code files
- **Chunking Engine** — Semantic chunking with heading awareness and configurable overlap
- **Embedding Generator** — OpenAI ada-002 with interface for alternatives
- **Vector Store** — ChromaDB for local dev, Pinecone for production (adapter pattern)

### Delivery Layer (Layer 4)
- **Browser Extension** — Chrome MV3 with content scripts for chatgpt.com and claude.ai, side panel for context preview
- **MCP Server** — Model Context Protocol server with tailor_context, search_docs, upload_document tools
- **Custom GPT** — GPT Store listing with Actions calling the tailoring API
- **REST API** — Public API for any integration
- **Landing Page** — Marketing site with hero, features, pricing, and footer (React + Vite + Tailwind)

## Technology Choices

| Choice | Rationale |
|--------|-----------|
| Express + TypeScript | Lightweight, flexible API server with type safety |
| PostgreSQL + Prisma | Relational data with type-safe ORM and migrations |
| ChromaDB / Pinecone | Local-first vector DB with production-ready cloud option |
| Redis | Fast caching for rate limiting and frequently-accessed chunks |
| Bull | Reliable job queue for async document processing |
| React + Vite | Fast dashboard with HMR for the management UI |
| Chrome MV3 | Modern extension API with service workers and side panels |
| Zod | Runtime validation with TypeScript type inference |

## Deployment

### Local Development
- Docker Compose: PostgreSQL + Redis + ChromaDB
- Express server with ts-node
- Vite dev server for dashboard
- Chrome extension loaded unpacked

### Production
- API: Railway (Dockerfile multi-stage build, health check at /health)
- Database: Managed PostgreSQL
- Vectors: Pinecone cloud
- Cache: Managed Redis
- Auth: Clerk
- Extension: Chrome Web Store
- CI: GitHub Actions (lint → typecheck → test → docker build)
