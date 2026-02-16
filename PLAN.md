# AgentTailor — Master Plan

## Vision

AgentTailor is a cross-platform add-on that makes AI agents (ChatGPT and Claude) dramatically better at any task by automatically assembling the optimal context. Instead of users manually crafting prompts and copy-pasting from docs, AgentTailor analyzes the task, retrieves relevant information from multiple sources, and injects a tailored context package into the AI's conversation.

### Key Differentiators
1. **Cross-platform** — Works for both GPT and Claude (browser extension, MCP, Custom GPT)
2. **Context quality over quantity** — Smart compression beats token stuffing
3. **Transparency** — Always shows WHAT context was assembled and WHY
4. **Speed** — Aggressive caching, pre-computation where possible
5. **Privacy** — Documents never leave chosen storage; option for fully local processing

## Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Backend | Node.js + Express + TypeScript | API server, document processing, intelligence engine |
| Database | PostgreSQL (Prisma ORM) | User data, project metadata, document records |
| Vector Storage | ChromaDB (local) / Pinecone (prod) | Embedding storage and similarity search |
| Cache | Redis | Frequently-accessed context chunks, session state |
| Queue | Bull | Async document processing jobs |
| Dashboard | React + TypeScript + Vite + Tailwind | Upload docs, manage projects, view context logs |
| Extension | Chrome Manifest V3 | Content scripts for chatgpt.com and claude.ai |
| MCP Server | Model Context Protocol | Native Claude Desktop/Code integration |
| Validation | Zod | Schema-first validation with inferred types |
| Testing | Vitest | Unit, integration, and component tests |

## Core Architecture — 4 Layers

### Layer 1: Context Gathering (Data Ingestion)
- Document upload & processing pipeline (PDF, DOCX, MD, TXT, code files, URLs)
- Semantic chunking with configurable overlap (NOT fixed-size)
- Embedding generation using OpenAI ada-002
- Vector storage (ChromaDB local, Pinecone cloud)
- Web search module (Tavily API, Brave Search fallback)

### Layer 2: Context Intelligence Engine (THE CORE)
- **Task Analyzer** — Parse user input → knowledge domains, task type, complexity
- **Relevance Scorer** — Cross-encoder reranking (not just cosine similarity)
- **Gap Detector** — Identify insufficient context, trigger web search
- **Context Compressor** — Hierarchy: exact quotes (high), summaries (medium), keywords (low)
- **Source Synthesizer** — Merge docs + web + APIs, resolve contradictions
- **Priority Ranker** — Rank by relevance, recency, authority, specificity
- **Context Window Manager** — Track tokens, enforce model limits (128K GPT-4, 200K Claude)

### Layer 3: Agent Configuration
- **Platform Formatter** — Format context for GPT (structured system messages) vs Claude (XML-tagged blocks)
- **Dynamic Prompt Generator** — Optimized system prompts per platform

### Layer 4: Delivery & Integration
- **Browser Extension** — Chrome extension for chatgpt.com and claude.ai with sidebar preview
- **MCP Server** — Native Claude Desktop/Code integration via Model Context Protocol
- **Custom GPT** — GPT Store listing with Actions calling AgentTailor API
- **REST API** — Backend API for any client

## Key Features

- **Smart Context Assembly** — Automatically gathers and ranks relevant information
- **Document Ingestion** — Upload PDF, DOCX, MD, TXT, code files
- **Web Search Integration** — Fill knowledge gaps with Tavily/Brave search
- **Cross-Platform Delivery** — Browser extension, MCP server, Custom GPT, REST API
- **Transparency** — See what context was assembled and why each piece was included
- **Quality Scoring** — Score tailored outputs on coverage, diversity, relevance
- **Citation Tracking** — Every context chunk traced to its source

## Phase Overview

| Phase | Theme | Tasks | Key Deliverables |
|-------|-------|-------|-----------------|
| 1 | Foundation | T001–T008 (8) | Monorepo, DB, auth, doc pipeline, vector search |
| 2 | Intelligence Engine | T009–T015 (7) | Task analyzer, scorer, compressor, /api/tailor |
| 3 | Web Search | T016–T018 (3) | Tavily/Brave integration, gap-triggered search, citations |
| 4 | Dashboard | T019–T024 (6) | React app, project management, doc upload UI, session viewer |
| 5 | Browser Extension | T025–T030 (6) | Chrome extension, ChatGPT/Claude detection, context injection |
| 6 | MCP Server | T031–T033 (3) | Claude Desktop integration, tools, resources |
| 7 | Custom GPT | T034–T035 (2) | GPT Actions, GPT Store listing |
| 8 | Polish & Launch | T036–T040 (5) | Quality scoring, analytics, rate limiting, landing page, tests |

**Total: 40 tasks across 8 phases**

## Data Model

### Core Entities
- **User** — id, email, name, plan, settings
- **Project** — id, user_id, name, description, created_at
- **Document** — id, project_id, filename, type, status, chunk_count, metadata
- **Chunk** — id, document_id, content, embedding, position, metadata
- **TailoringSession** — id, user_id, project_id, task_input, assembled_context, target_platform, token_count, quality_score
- **WebSearchResult** — id, session_id, query, url, title, snippet, relevance_score

## Non-Functional Requirements

- Tailoring response: < 3s cached, < 8s with web search
- Documents up to 50MB, up to 1000 chunks per project
- Extension: < 100ms page load impact
- Encrypted at rest, GDPR-compliant
- Rate limits: Free 50/day, Pro 500/day, Team 2000/day

## Architecture

See [System Overview](./architecture/system-overview.md) for detailed architecture documentation.

## Task Board

See [TASK_BOARD.md](./TASK_BOARD.md) for the complete task breakdown with dependency tracking.
