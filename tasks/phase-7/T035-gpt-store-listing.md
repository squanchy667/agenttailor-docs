# T035 — GPT Store Listing Preparation

| Field | Value |
|-------|-------|
| Phase | 7 — Custom GPT |
| Agent Type | integration-agent |
| Depends On | T034 |
| Blocks | T040 |
| Branch | `feat/T035-gpt-store-listing` |
| Status | PENDING |

## Objective
Prepare the Custom GPT for GPT Store publication by writing system instructions that explain AgentTailor's capabilities, defining conversation starters for common use cases, creating a privacy policy, and adding a health endpoint for GPT Builder testing.

## Context
The GPT Store listing is the first impression for ChatGPT users discovering AgentTailor. The system instructions are critical — they teach the Custom GPT how to use AgentTailor's API effectively, when to call which endpoint, and how to present results to users. Poor instructions lead to a frustrating user experience even if the API works perfectly. Conversation starters reduce friction for first-time users by demonstrating common workflows. The privacy policy is mandatory for GPT Store publication.

## Subtasks
1. Create `server/openapi/gpt-instructions.md` — System instructions for the Custom GPT:
   - Identity: "You are an AI assistant enhanced with AgentTailor, a context assembly engine that gives you access to the user's project documentation and web search results."
   - Capabilities: explain what `tailor_context`, `search_docs`, and `list_projects` do.
   - Workflow guidance: when the user asks a project-related question, first list their projects (if projectId unknown), then call tailor_context with a well-crafted task description, then answer using the returned context.
   - Formatting: always cite sources when using tailored context, indicate quality score if available.
   - Limitations: explain that context is assembled from user's uploaded docs and web search, not fabricated.
   - Error handling: if an API call fails, explain what happened and suggest alternatives.
2. Create `server/openapi/gpt-metadata.json` — GPT configuration metadata:
   - `name`: "AgentTailor"
   - `description`: Concise pitch (under 300 chars) for GPT Store listing.
   - `conversation_starters`: array of 4 starters:
     - "Help me understand my project's architecture"
     - "Search my docs for authentication implementation details"
     - "Assemble context for building a new API endpoint"
     - "What projects do I have in AgentTailor?"
   - `capabilities`: { "web_browsing": false, "code_interpreter": false, "dall_e": false } — relies on AgentTailor Actions only.
3. Create `docs/privacy-policy.md` — Privacy policy for GPT Store:
   - What data is collected: task descriptions sent to the API, document content uploaded by users.
   - How data is stored: encrypted at rest, user-scoped, deleted on account deletion.
   - Third-party services: OpenAI (embeddings), search providers (web search).
   - User rights: data export, deletion, access to their data via dashboard.
   - Contact information placeholder.
4. Add health endpoint to `server/src/routes/gptActions.ts`:
   - `GET /gpt/health` — Returns `{ status: "ok", version: "x.y.z", tools: ["tailor", "search", "projects"] }`.
   - No authentication required (GPT Builder uses this to verify connectivity).
5. Verify end-to-end: ensure the OpenAPI spec from T034 references the correct server URL, instructions reference the correct tool names, and conversation starters map to real API capabilities.

## Files to Create/Modify
- `server/openapi/gpt-instructions.md` — System instructions for the Custom GPT
- `server/openapi/gpt-metadata.json` — Name, description, conversation starters, capabilities
- `docs/privacy-policy.md` — Privacy policy for GPT Store compliance
- `server/src/routes/gptActions.ts` — Add /gpt/health endpoint (modify)

## Acceptance Criteria
- [ ] System instructions clearly explain how the GPT should use each AgentTailor action
- [ ] Instructions include workflow guidance (list projects, then tailor, then answer)
- [ ] Conversation starters cover the 4 main use cases (architecture, search, context, project list)
- [ ] GPT metadata JSON is well-formed with all required fields
- [ ] Privacy policy covers data collection, storage, third parties, and user rights
- [ ] `GET /gpt/health` returns 200 with status, version, and available tools
- [ ] Health endpoint does not require authentication
- [ ] All tool names in instructions match the OpenAPI spec operation IDs
