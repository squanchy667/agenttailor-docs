# AgentTailor

**Cross-platform AI agent tailoring add-on for ChatGPT and Claude**

## Vision

AgentTailor makes AI agents dramatically better at any task by automatically assembling the optimal context. Instead of users manually crafting prompts and copy-pasting from docs, AgentTailor analyzes the task, retrieves relevant information from multiple sources (uploaded documents, web search, APIs), and injects a tailored context package into the AI's conversation.

It works across both ChatGPT and Claude via browser extension, MCP server, and Custom GPT — making context portability the core moat.

## Architecture at a Glance

```
┌─────────────────────────────────────────────────────┐
│                   Delivery Layer                     │
│  Browser Extension │ MCP Server │ Custom GPT │ API  │
└────────────┬───────────────────────────┬────────────┘
             │                           │
┌────────────▼───────────────────────────▼────────────┐
│              Context Intelligence Engine             │
│  Task Analyzer → Relevance Scorer → Gap Detector    │
│  Context Compressor → Source Synthesizer → Ranker   │
│  Context Window Manager → Platform Formatter        │
└────────────┬───────────────────────────┬────────────┘
             │                           │
┌────────────▼──────────┐  ┌────────────▼────────────┐
│   Context Gathering    │  │     Web Search          │
│  Doc Upload → Extract  │  │  Tavily / Brave API     │
│  Chunk → Embed → Store │  │  Result Processing      │
│  Vector Search         │  │  Citation Tracking      │
└────────────────────────┘  └─────────────────────────┘
             │                           │
┌────────────▼───────────────────────────▼────────────┐
│                   Data Layer                         │
│  PostgreSQL │ ChromaDB/Pinecone │ Redis │ Bull Queue │
└─────────────────────────────────────────────────────┘
```

See [System Overview](./architecture/system-overview.md) for the full architecture.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Backend | Node.js, Express.js, TypeScript |
| Database | PostgreSQL (Prisma ORM) |
| Vector Storage | ChromaDB (local), Pinecone (production) |
| Cache | Redis |
| Queue | Bull (async document processing) |
| Dashboard | React, TypeScript, Vite, Tailwind CSS |
| Browser Extension | Chrome Manifest V3 |
| MCP Server | Model Context Protocol for Claude Desktop/Code |
| Validation | Zod |
| Testing | Vitest |

## Target Users

1. **Developers** — Use ChatGPT/Claude for coding with project docs, READMEs, architecture docs
2. **Knowledge Workers** — Consultants, researchers, analysts who need AI with domain docs
3. **Teams** — Shared context libraries so every member gets equally good AI results

## Repos

| Repo | Purpose |
|------|---------|
| `agenttailor-docs` | Architecture, specs, task board, test plans |
| `agenttailor` | Source code (backend, dashboard, extension, MCP server) |

## Documentation

See [SUMMARY.md](./SUMMARY.md) for the full table of contents.

## Task Board

See [TASK_BOARD.md](./TASK_BOARD.md) for the complete task breakdown with agent assignments and parallel execution plan.
