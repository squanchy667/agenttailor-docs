# State Management

## Approach

- Zod schemas as single source of truth for data shapes
- Server-side state in PostgreSQL + Redis + vector stores
- Dashboard state via React Query for server data
- Extension state via Chrome storage API

## Server State

### PostgreSQL (persistent)
- Users, projects, documents, tailoring sessions, web search results
- Managed via Prisma ORM with typed queries
- Migrations for schema evolution

### Redis (ephemeral)
- Rate limit counters per user
- Cached context chunks for frequently-accessed projects
- Session state for active tailoring operations
- Bull job queue metadata

### Vector Store (embeddings)
- ChromaDB for local development
- Pinecone for production
- Document chunk embeddings with metadata (doc_id, position, headings)

## Dashboard State

Managed with React Query (@tanstack/react-query):
- Automatic caching and refetching
- Optimistic updates for mutations
- Loading and error states built-in

```tsx
const { data, isLoading, error } = useQuery({
  queryKey: ['projects', projectId],
  queryFn: () => fetchProject(projectId),
});
```

## Extension State

- **Chrome Storage API** — Persistent settings (active project, API endpoint, preferences)
- **Runtime Messages** — Communication between content scripts, service worker, and side panel
- **In-memory** — Current tailoring state, preview content
