# Features

## Core Features (v1.0)

### Context Intelligence Engine
- **Smart Context Assembly** — Automatically analyzes tasks, retrieves relevant info from documents and web, assembles optimal context packages
- **7-Stage Pipeline** — Task Analyzer, Relevance Scorer, Gap Detector, Context Compressor, Source Synthesizer, Priority Ranker, Context Window Manager
- **Graceful Degradation** — Every pipeline stage has try/catch with meaningful fallbacks. The engine never returns empty results
- **Cross-Encoder Reranking** — Goes beyond cosine similarity with cross-encoder scoring for higher accuracy
- **Hierarchical Compression** — 4 levels (FULL > SUMMARY > KEYWORDS > REFERENCE) based on relevance score thresholds

### Document Management
- **Document Ingestion** — Upload PDF, DOCX, MD, TXT, and code files with semantic chunking and embedding
- **Semantic Chunking** — Heading-aware splitting with configurable overlap, not fixed-size blocks
- **Vector Search** — ChromaDB (local) / Pinecone (production) with adapter pattern for easy switching

### Web Search
- **Gap-Triggered Search** — Automatically detects insufficient context coverage and triggers web search
- **Dual Provider** — Tavily API (primary) with Brave Search fallback
- **Citation Tracking** — Every chunk traced to its source document, page, heading, or web URL

### Delivery Platforms
- **Browser Extension** — Chrome MV3 with content scripts for chatgpt.com and claude.ai, side panel for context preview and editing
- **MCP Server** — Native Claude Desktop/Code integration via Model Context Protocol with 4 tools and 2 resource providers
- **Custom GPT** — GPT Store listing with OpenAPI Actions calling the tailoring API
- **REST API** — Public API for any integration

### Dashboard
- **Project Management** — Create, organize, and manage projects with document libraries
- **Document Upload UI** — Drag-and-drop with processing status tracking
- **Tailoring Session Viewer** — View context assembly history with source attribution
- **Usage Analytics** — Session trends, quality trends, project stats, plan usage with charts
- **Settings** — API key management, context preferences, plan management

### Quality & Operations
- **Quality Scoring** — 4 weighted sub-scores: coverage (0.35), relevance (0.30), diversity (0.20), compression (0.15). Composite 0-100 with actionable suggestions
- **Rate Limiting** — Redis INCR+EXPIRE with fail-open. X-RateLimit headers. Plan-based tiers (FREE 50/day, PRO 500/day, TEAM 2000/day)
- **Plan Enforcement** — Structural limits: max projects, max docs per project, max file size per plan tier
- **Transparency** — Every context piece shows its source, relevance score, and why it was included
- **Platform-Specific Formatting** — XML tags for ChatGPT, Markdown + `<document>` tags for Claude

## Planned Features

See the [Roadmap](./roadmap.md) for the full feature pipeline. Highlights:

- **CLI tool** — Context assembly from the terminal
- **VS Code / JetBrains extensions** — IDE-native context panels
- **Plugin ecosystem** — Source plugins (Notion, Confluence, JIRA), embedding plugins (Cohere, local models), platform plugins (Gemini, Cursor)
- **Code-aware chunking** — AST-based splitting for programming languages
- **GitHub repo ingestion** — Auto-index a repository's docs and code
- **Team workspaces** — Shared projects with role-based access
- **Self-hosted enterprise** — Helm chart, air-gapped mode, compliance features
- **Context strategies** — Predefined and custom strategies per project type
- **Feedback learning** — Improve scoring based on what context users actually use
