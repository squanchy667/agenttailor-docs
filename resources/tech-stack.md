# Tech Stack

## Technology Choices

| Technology | Version | Purpose |
|-----------|---------|---------|
| Node.js | ^20 | Runtime |
| Express.js | ^4 | API server |
| TypeScript | ^5 | Type safety across all packages |
| PostgreSQL | ^16 | Relational data (users, projects, documents, sessions) |
| Prisma | ^5 | Type-safe ORM with migrations |
| ChromaDB | ^0.4 | Local vector storage for embeddings |
| Pinecone | ^2 | Cloud vector storage for production |
| Redis | ^7 | Caching, rate limiting, session state |
| Bull | ^4 | Job queue for async document processing |
| React | ^18 | Dashboard UI framework |
| Vite | ^5 | Build tool and dev server for dashboard |
| Tailwind CSS | ^3 | Utility-first CSS for dashboard and extension |
| React Query | ^5 | Server state management in dashboard |
| Clerk | latest | Authentication (social login + API keys) |
| Zod | ^3 | Runtime validation and schema definitions |
| Vitest | ^1 | Test runner |
| OpenAI SDK | ^4 | Embedding generation (ada-002) |
| Tavily SDK | latest | Web search API |
| Chrome MV3 | — | Browser extension platform |
| MCP SDK | latest | Model Context Protocol for Claude |

## Rationale

| Choice | Why |
|--------|-----|
| Express + TypeScript | Lightweight, flexible, well-suited for the multi-service backend |
| PostgreSQL + Prisma | Battle-tested relational DB with type-safe ORM and auto migrations |
| ChromaDB / Pinecone | Local-first dev with adapter pattern for production cloud vectors |
| Redis | Fast ephemeral storage for rate limits and cache — avoids DB load |
| Bull | Reliable queue on Redis — doc processing is inherently async |
| React + Vite | Fast dashboard dev with HMR, large ecosystem |
| Chrome MV3 | Modern extension API with service workers and side panels |
| Clerk | Quick auth setup with social login, minimal custom code |
| Zod | Schema-first: define once, validate everywhere, infer types |
