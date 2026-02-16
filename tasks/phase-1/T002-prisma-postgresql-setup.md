# T002 — PostgreSQL Setup with Prisma ORM and Migrations

| Field | Value |
|-------|-------|
| Phase | 1 — Foundation |
| Agent Type | backend-agent |
| Depends On | T001 |
| Blocks | T003, T004, T006, T007 |
| Branch | `feat/T002-prisma-postgresql-setup` |
| Status | PENDING |

## Objective
Configure Prisma ORM with PostgreSQL, create the initial schema with all core entities (User, Project, Document, Chunk, TailoringSession, WebSearchResult), and generate the first migration.

## Context
The database schema is the backbone of AgentTailor. Every feature — document ingestion, context assembly, session history — depends on these models. Using Prisma gives us type-safe queries, auto-generated client, and declarative migrations. Getting the schema right now avoids costly migrations later.

## Subtasks
1. Install Prisma dependencies in `server/` — `prisma` (dev), `@prisma/client`.
2. Run `npx prisma init` to generate `prisma/schema.prisma` and configure datasource for PostgreSQL.
3. Define the `User` model: `id` (UUID, default), `email` (unique), `name`, `clerkId` (unique, for Clerk integration), `plan` (enum: FREE, PRO, TEAM), `settings` (JSON), `createdAt`, `updatedAt`.
4. Define the `Project` model: `id` (UUID), `userId` (FK), `name`, `description` (optional), `createdAt`, `updatedAt`. Relation: User has many Projects.
5. Define the `Document` model: `id` (UUID), `projectId` (FK), `filename`, `type` (enum: PDF, DOCX, MARKDOWN, TXT, CODE), `status` (enum: PROCESSING, READY, ERROR), `chunkCount` (default 0), `metadata` (JSON), `sizeBytes` (Int), `createdAt`, `updatedAt`. Relation: Project has many Documents.
6. Define the `Chunk` model: `id` (UUID), `documentId` (FK), `content` (Text), `embedding` (unsupported for now — stored in vector DB, not Prisma), `position` (Int), `metadata` (JSON — headings, page_num, section), `tokenCount` (Int), `createdAt`. Relation: Document has many Chunks.
7. Define the `TailoringSession` model: `id` (UUID), `userId` (FK), `projectId` (FK), `taskInput` (Text), `assembledContext` (Text), `targetPlatform` (enum: CHATGPT, CLAUDE), `tokenCount` (Int), `qualityScore` (Float, optional), `createdAt`. Relations: User has many Sessions, Project has many Sessions.
8. Define the `WebSearchResult` model: `id` (UUID), `sessionId` (FK), `query`, `url`, `title`, `snippet` (Text), `relevanceScore` (Float), `createdAt`. Relation: TailoringSession has many WebSearchResults.
9. Add database indexes on frequently queried columns: `Document.projectId`, `Chunk.documentId`, `TailoringSession.userId`, `TailoringSession.projectId`.
10. Create `server/src/lib/prisma.ts` — Prisma client singleton (prevents multiple instances in dev with hot reload).
11. Create `server/src/lib/db.ts` — Thin helper layer with typed query helpers for common operations (findUserByClerkId, getProjectWithDocCount, etc.).
12. Run `npx prisma migrate dev --name init` to generate and apply the initial migration.
13. Add npm scripts: `db:migrate`, `db:push`, `db:studio`, `db:seed`.

## Files to Create/Modify
- `server/prisma/schema.prisma` — Full Prisma schema with all 6 models, enums, and indexes
- `server/prisma/migrations/` — Auto-generated initial migration SQL
- `server/src/lib/prisma.ts` — Prisma client singleton with logging config
- `server/src/lib/db.ts` — Typed helper queries for common database operations
- `server/package.json` — Add prisma dependencies and db scripts

## Acceptance Criteria
- [ ] `npx prisma migrate dev` runs successfully against Docker PostgreSQL
- [ ] `npx prisma generate` produces typed client with all models
- [ ] Prisma Studio (`npx prisma studio`) opens and shows all 6 tables
- [ ] Prisma client singleton prevents multiple instances during hot reload
- [ ] All model relations are correctly defined (User->Projects, Project->Documents, etc.)
- [ ] Indexes exist on foreign key columns for query performance
- [ ] Enums (Plan, DocumentType, DocumentStatus, TargetPlatform) are properly defined
- [ ] `db.ts` helpers are type-safe and compile without errors
