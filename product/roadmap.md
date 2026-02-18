# Roadmap

## Completed Milestones (v1.0)

| Milestone | Description | Tasks | Status |
|-----------|-------------|-------|--------|
| M1 — Foundation | Backend scaffold, DB, auth, doc processing pipeline | T001–T008 | DONE |
| M2 — Intelligence Engine | Core context intelligence: analyzer, scorer, compressor, /api/tailor | T009–T015 | DONE |
| M3 — Web Search | Tavily/Brave integration, gap-triggered search, citations | T016–T018 | DONE |
| M4 — Dashboard | React management UI: projects, documents, sessions, settings | T019–T024 | DONE |
| M5 — Browser Extension | Chrome extension for ChatGPT and Claude with context injection | T025–T030 | DONE |
| M6 — MCP Server | Native Claude Desktop/Code integration via Model Context Protocol | T031–T033 | DONE |
| M7 — Custom GPT | GPT Store listing with Actions for context retrieval | T034–T035 | DONE |
| M8 — Launch | Quality scoring, analytics, rate limiting, landing page, tests, CI/CD | T036–T040 | DONE |

**v1.0 delivered: 40/40 tasks, 8/8 phases, 42 commits, 38 tests passing.**

---

## v1.1 — Local Mode (DONE)

Zero-config setup for open-source adoption. T041–T052.

| Feature | Description | Status |
|---------|-------------|--------|
| Local auth mode | `AUTH_MODE=local` — no Clerk, auto-created local user | DONE |
| Local embeddings | `EMBEDDING_PROVIDER=local` — @xenova/transformers, Xenova/all-MiniLM-L6-v2 | DONE |
| Noop cross-encoder | Cosine-only scoring, no Cohere/OpenAI API needed | DONE |
| Setup script | `npm run setup` — docker, install, db push, seed | DONE |
| Seed data | Default user + "Getting Started" project | DONE |
| MCP port fix | Default `localhost:4000` matches actual server port | DONE |
| .env.local.example | Minimal env for local mode | DONE |
| Smoke tests | 6 new tests (162 total passing) | DONE |

---

## Future Roadmap

### v1.2 — Publish & Distribute (Next)

Ship to npm, Docker Hub, and MCP registry so people can use it worldwide.

| Feature | Description | Priority |
|---------|-------------|----------|
| npm publish `agent-tailor-mcp` | `npx agent-tailor-mcp` just works as MCP server | P0 |
| Docker Compose one-liner | `docker compose up` starts full stack (server + db + redis + chroma) | P0 |
| GitHub release | Tagged release with changelog, install instructions, GIF demo | P0 |
| MCP Registry listing | Submit to official MCP server directory | P0 |
| Chrome Web Store | Publish extension for ChatGPT + Claude context injection | P0 |
| GPT Store listing | Publish Custom GPT with Actions pointing at hosted/self-hosted API | P1 |
| OpenAPI docs | Swagger UI at `/api/docs` for interactive API exploration | P1 |
| Homebrew formula | `brew install agenttailor` for Mac users | P2 |

### v1.3 — Developer Tools

Expand beyond browser — meet developers where they work.

| Feature | Description | Priority |
|---------|-------------|----------|
| CLI tool | `agenttailor context "implement auth"` — context from terminal | P0 |
| VS Code extension | Context panel in the editor, auto-detect project | P0 |
| JetBrains plugin | IntelliJ/WebStorm integration via the REST API | P1 |
| Cursor/Windsurf integration | MCP-based or API-based hooks for AI-native editors | P1 |
| GitHub repo ingestion | Point at a repo URL, auto-index README, docs, and code | P0 |
| Git watch mode | Auto-re-index when tracked files change | P2 |

### v1.4 — Plugin Ecosystem

Extensible architecture so anyone can add sources, platforms, and models.

| Feature | Description | Priority |
|---------|-------------|----------|
| Source plugins | Notion, Confluence, JIRA, Slack, Google Docs, Linear, Figma | P0 |
| Platform plugins | Gemini, Copilot Chat, Perplexity, any LLM web UI | P1 |
| Embedding plugins | Cohere, Voyage AI, local sentence-transformers, Ollama | P0 |
| Search plugins | Serper, SerpAPI, Google Custom Search, custom endpoints | P2 |
| Plugin SDK | `@agenttailor/plugin-sdk` with typed interfaces and examples | P0 |
| Plugin marketplace | Community-contributed plugins discoverable from dashboard | P2 |

### v1.5 — Intelligence Upgrades

Make the context engine smarter and more adaptive.

| Feature | Description | Priority |
|---------|-------------|----------|
| Code-aware chunking | AST-based splitting by function/class/module for programming languages | P0 |
| Context strategies | Predefined strategies per project type (coding, legal, research, medical) | P1 |
| Feedback learning | Track which context pieces users kept/removed, improve scoring | P1 |
| Multi-modal context | Support images, diagrams, and charts as context sources | P2 |
| Streaming assembly | Real-time context preview as each pipeline stage completes | P1 |
| Context versioning | Snapshot and compare context strategies over time | P2 |
| A/B testing | Compare context strategies and measure quality score differences | P2 |

### v1.6 — Teams & Collaboration

Make AgentTailor valuable for organizations, not just individuals.

| Feature | Description | Priority |
|---------|-------------|----------|
| Team workspaces | Shared projects and knowledge bases across team members | P0 |
| Role-based access | Admin, Editor, Viewer roles with granular permissions | P0 |
| Context templates | Reusable "when someone asks about X, always include Y" rules | P1 |
| Organization settings | Org-wide defaults, approved sources, blocked domains | P1 |
| Audit trail | Who accessed what context, when, and for what task | P1 |
| SSO | SAML 2.0 and OAuth for enterprise identity providers | P2 |
| Usage reports | Per-team and per-user analytics, exportable CSV | P2 |

### v1.7 — Enterprise & Self-Hosted

Production-grade deployment for organizations with strict requirements.

| Feature | Description | Priority |
|---------|-------------|----------|
| Helm chart | Kubernetes deployment with horizontal scaling | P0 |
| Air-gapped mode | Local embeddings (sentence-transformers) + no external calls | P0 |
| Data retention policies | Auto-delete sessions/documents after configurable period | P1 |
| Compliance | SOC 2 readiness, GDPR data export/deletion, HIPAA considerations | P1 |
| Custom embedding models | Bring your own model via OpenAI-compatible API | P1 |
| Webhooks | POST notifications for document processed, session created, etc. | P1 |
| Stripe billing | Usage-based billing with plan upgrades via dashboard | P2 |
| Multi-region | Deploy across regions for data residency requirements | P2 |

### v2.0 — Platform

Long-term vision: AgentTailor becomes the standard context layer for AI agents.

| Feature | Description |
|---------|-------------|
| Context API standard | Open specification for context retrieval that any AI tool can implement |
| Agent-to-agent context | AI agents share and request context from each other |
| Real-time collaboration | Multiple users building context for the same session |
| Context marketplace | Buy/sell domain-specific knowledge packages |
| Fine-tuned rankers | Train custom relevance models on your domain data |
| Mobile companion | iOS/Android app for context management on the go |

---

## Contributing

We welcome contributions at every level — from fixing typos to building new platform plugins. See [CONTRIBUTING.md](../CONTRIBUTING.md) in the code repo for how to get started.

### Good First Issues

- Add a new embedding provider (Cohere, Voyage AI)
- Add a new search provider (Serper, Google Custom Search)
- Improve code chunking with AST-based splitting for JavaScript/TypeScript
- Add Firefox extension support (WebExtensions API)
- Add dark mode to the dashboard
- Improve test coverage for intelligence engine modules
- Add i18n support (translation framework)
