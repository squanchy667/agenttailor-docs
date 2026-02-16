# Setup Guide

## Prerequisites

- Node.js >= 20
- npm >= 10
- Docker and Docker Compose (for PostgreSQL, Redis, ChromaDB)
- Chrome browser (for extension development)

## Installation

```bash
git clone <repo-url> agenttailor
cd agenttailor
npm install
```

## Environment Variables

```bash
cp .env.example .env
```

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://postgres:postgres@localhost:5432/agenttailor` |
| `REDIS_URL` | Redis connection string | `redis://localhost:6379` |
| `CHROMA_URL` | ChromaDB URL | `http://localhost:8000` |
| `OPENAI_API_KEY` | OpenAI API key (for embeddings) | — |
| `TAVILY_API_KEY` | Tavily API key (for web search) | — |
| `CLERK_SECRET_KEY` | Clerk auth secret | — |
| `CLERK_PUBLISHABLE_KEY` | Clerk auth public key | — |
| `PORT` | Server port | `3001` |
| `VITE_API_URL` | API base URL for dashboard | `http://localhost:3001/api` |

## Running Locally

```bash
# Start infrastructure (PostgreSQL, Redis, ChromaDB)
docker compose up -d

# Run database migrations
npx prisma migrate dev --schema server/prisma/schema.prisma

# Start server + dashboard in dev mode
npm run dev

# Load browser extension
# 1. Open chrome://extensions
# 2. Enable "Developer mode"
# 3. Click "Load unpacked" → select extension/dist/
```

## Project Structure

```
agenttailor/
├── server/                  # Express API server
│   ├── prisma/              # Prisma schema and migrations
│   └── src/
│       ├── routes/          # API route handlers
│       ├── services/        # Business logic
│       │   ├── intelligence/ # Context intelligence engine
│       │   ├── chunking/    # Document chunking
│       │   ├── embedding/   # Embedding generation
│       │   ├── vectorStore/ # Vector DB adapters
│       │   ├── search/      # Web search integration
│       │   └── textExtractor/ # Document text extraction
│       ├── middleware/      # Auth, rate limiting, validation
│       ├── workers/         # Bull job workers
│       └── lib/             # Shared utilities (prisma, redis, queue)
├── dashboard/               # React dashboard (Vite)
│   └── src/
│       ├── components/      # UI components
│       ├── pages/           # Route pages
│       ├── hooks/           # Custom React hooks
│       └── lib/             # API client, query config
├── extension/               # Chrome Extension (Manifest V3)
│   └── src/
│       ├── background/      # Service worker
│       ├── content/         # Content scripts (chatgpt, claude)
│       ├── sidepanel/       # Side panel UI
│       └── popup/           # Extension popup
├── mcp-server/              # MCP Server for Claude
│   └── src/
│       ├── tools/           # MCP tool implementations
│       └── resources/       # MCP resource providers
├── shared/                  # Shared types and Zod schemas
│   └── src/schemas/
├── docker-compose.yml       # Local dev infrastructure
└── package.json             # Workspace root
```

## Useful Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Start server + dashboard in dev mode |
| `npm run dev:server` | Start server only |
| `npm run dev:dashboard` | Start dashboard only |
| `npm run build` | Build all packages |
| `npm run typecheck` | TypeScript type checking |
| `npm run test` | Run tests |
| `npm run lint` | Lint code |
| `npx prisma studio` | Open Prisma database browser |
| `npx prisma migrate dev` | Run pending migrations |
