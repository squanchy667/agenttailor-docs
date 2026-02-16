# T034 — GPT Action Schema and API Adaptations

| Field | Value |
|-------|-------|
| Phase | 7 — Custom GPT |
| Agent Type | integration-agent |
| Depends On | T015 |
| Blocks | T035 |
| Branch | `feat/T034-gpt-action-schema-api` |
| Status | PENDING |

## Objective
Create an OpenAI GPT Action schema (OpenAPI 3.1 spec) for the tailoring API and implement GPT-compatible API endpoints that work within GPT Actions constraints, including API key authentication and simplified request/response formats.

## Context
GPT Actions are how Custom GPTs call external APIs. They require a valid OpenAPI spec and have specific constraints: no streaming, simple auth (API key or OAuth), responses must be concise enough for GPT to process, and endpoints should be self-documenting. By creating a dedicated `/gpt/` route namespace, we avoid polluting the main API with GPT-specific adaptations while reusing the core service layer. This integration opens AgentTailor to ChatGPT's massive user base and the GPT Store marketplace.

## Subtasks
1. Create `shared/src/schemas/gptActions.ts` — Zod schemas for GPT Action endpoints:
   - `GptTailorRequest` — task (string, max 2000 chars), projectId (string UUID), maxTokens (number, optional, default 3000).
   - `GptTailorResponse` — context (string), sourceCount (number), topSources (array of { title, type }), qualityScore (number).
   - `GptSearchRequest` — query (string, max 500 chars), projectId (string UUID), topK (number, optional, default 3).
   - `GptSearchResponse` — results (array of { content: string shortened to 500 chars, source: string, relevance: number }).
   - `GptProjectListResponse` — projects (array of { id, name, documentCount }).
2. Create `server/src/middleware/gptAuth.ts` — API key authentication middleware for GPT Actions:
   - Read `X-Api-Key` header (GPT Actions convention).
   - Look up API key in database, resolve to user.
   - Attach user to request context (same shape as Clerk auth).
   - Return 401 with `{ error: "Invalid API key" }` for missing or invalid keys.
3. Create `server/src/routes/gptActions.ts` — GPT-compatible endpoints:
   - `POST /gpt/tailor` — Accept GptTailorRequest, call tailorOrchestrator, return GptTailorResponse (simplified, no raw chunks).
   - `POST /gpt/search` — Accept GptSearchRequest, call searchService, return GptSearchResponse (truncated content for token efficiency).
   - `GET /gpt/projects` — Return GptProjectListResponse (project list without full details).
   - All endpoints use `gptAuth` middleware.
   - All responses include `_help` field with brief usage guidance for GPT.
4. Create `server/openapi/gpt-actions.yaml` — OpenAPI 3.1 specification:
   - Info: title "AgentTailor", description explaining the service, version from package.json.
   - Servers: configurable base URL via environment variable.
   - Security: API key in `X-Api-Key` header.
   - Paths: `/gpt/tailor`, `/gpt/search`, `/gpt/projects` with full request/response schemas.
   - Components: reusable schemas matching Zod definitions.
   - Ensure spec passes OpenAPI validation (no $ref errors, valid examples).
5. Mount GPT routes in `server/src/routes/index.ts` under `/gpt` prefix.

## Files to Create/Modify
- `server/src/routes/gptActions.ts` — GPT-compatible API endpoints
- `server/openapi/gpt-actions.yaml` — OpenAPI 3.1 spec for GPT Store import
- `server/src/middleware/gptAuth.ts` — API key authentication middleware
- `shared/src/schemas/gptActions.ts` — Zod schemas for GPT Action I/O
- `server/src/routes/index.ts` — Mount GPT routes (modify)

## Acceptance Criteria
- [ ] OpenAPI spec validates successfully with an OpenAPI 3.1 validator
- [ ] `POST /gpt/tailor` returns simplified tailoring response suitable for GPT consumption
- [ ] `POST /gpt/search` returns truncated search results (each under 500 chars)
- [ ] `GET /gpt/projects` lists user's projects with document counts
- [ ] API key authentication works via `X-Api-Key` header
- [ ] Invalid or missing API key returns 401 with descriptive error
- [ ] All GPT responses include `_help` field with usage guidance
- [ ] OpenAPI spec can be imported into GPT Builder without errors
- [ ] Response payloads are compact enough for GPT to process (under 4000 tokens typical)
