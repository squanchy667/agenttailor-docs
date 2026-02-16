# T004 — Project CRUD API

| Field | Value |
|-------|-------|
| Phase | 1 — Foundation |
| Agent Type | backend-agent |
| Depends On | T002, T003 |
| Blocks | T007, T021 |
| Branch | `feat/T004-project-crud-api` |
| Status | PENDING |

## Objective
Implement RESTful endpoints for project management (create, read, update, delete, list) with Zod validation, auth-scoped queries, and pagination.

## Context
Projects are the primary organizational unit in AgentTailor. Users create projects to group related documents and tailoring sessions. Every document upload and context assembly happens within a project scope. This CRUD layer is the first feature endpoint and sets the pattern for all subsequent API routes.

## Subtasks
1. Create `shared/src/schemas/project.ts` — Zod schemas for:
   - `CreateProjectInput` — name (string, 1-100 chars), description (optional string, max 500 chars).
   - `UpdateProjectInput` — partial of CreateProjectInput.
   - `ProjectResponse` — full project object with id, name, description, documentCount, createdAt, updatedAt.
   - `ProjectListResponse` — array of ProjectResponse with pagination metadata (total, page, limit).
2. Create `server/src/services/projectService.ts` — Service layer with methods:
   - `createProject(userId, data)` — Creates project, returns with document count.
   - `getProject(userId, projectId)` — Fetches project scoped to user, throws 404 if not found or not owned.
   - `listProjects(userId, page, limit)` — Paginated list with document counts.
   - `updateProject(userId, projectId, data)` — Updates project fields, returns updated record.
   - `deleteProject(userId, projectId)` — Deletes project and cascades to documents, chunks, sessions.
3. Create `server/src/routes/projects.ts` — Express router with endpoints:
   - `POST /api/projects` — Create project. Validate body with Zod. Return 201.
   - `GET /api/projects` — List user's projects. Query params: `page`, `limit`, `search`.
   - `GET /api/projects/:id` — Get single project with document count and recent session count.
   - `PUT /api/projects/:id` — Update project. Validate body with Zod.
   - `DELETE /api/projects/:id` — Delete project. Return 204.
4. Add error handling: consistent error response shape `{ error: string, details?: object }`, 400 for validation, 404 for not found, 403 for unauthorized access.
5. Mount projects router in `server/src/routes/index.ts`.

## Files to Create/Modify
- `server/src/routes/projects.ts` — RESTful project endpoints
- `server/src/services/projectService.ts` — Project business logic and DB queries
- `shared/src/schemas/project.ts` — Zod schemas for project validation and response
- `server/src/routes/index.ts` — Mount projects router

## Acceptance Criteria
- [ ] `POST /api/projects` creates a project and returns 201 with the project object
- [ ] `GET /api/projects` returns paginated list of only the authenticated user's projects
- [ ] `GET /api/projects/:id` returns 404 for non-existent or other user's projects
- [ ] `PUT /api/projects/:id` updates only provided fields (partial update)
- [ ] `DELETE /api/projects/:id` removes the project and all associated documents/chunks
- [ ] Invalid input (empty name, name > 100 chars) returns 400 with Zod error details
- [ ] All endpoints require authentication (401 without valid token)
- [ ] Pagination metadata includes total count, current page, and limit
- [ ] Search query param filters projects by name (case-insensitive)
