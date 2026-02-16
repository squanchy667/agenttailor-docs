# T031 — MCP Server with tailor_context Tool

| Field | Value |
|-------|-------|
| Phase | 6 — MCP Server |
| Agent Type | integration-agent |
| Depends On | T015 |
| Blocks | T032, T033 |
| Branch | `feat/T031-mcp-server-tailor-context` |
| Status | PENDING |

## Objective
Implement a Model Context Protocol server that Claude Desktop and Claude Code can connect to, exposing a primary `tailor_context` tool that calls the tailoring pipeline and returns optimally assembled context for any task.

## Context
The MCP server is AgentTailor's native integration point for Claude products. Rather than requiring users to copy-paste context or use a browser extension, the MCP server lets Claude directly invoke the tailoring pipeline as a tool. This is the highest-value integration for Claude power users — it turns AgentTailor into an always-available context engine inside their IDE or desktop app. The MCP server runs as a separate process and communicates with the main backend via HTTP.

## Subtasks
1. Initialize `mcp-server/package.json` with dependencies: `@modelcontextprotocol/sdk`, `zod`, `node-fetch`. Add scripts for `dev` (ts-node-dev), `build` (tsc), and `start`.
2. Create `mcp-server/tsconfig.json` extending `../tsconfig.base.json` with output to `dist/`, targeting ES2022 with NodeNext module resolution.
3. Create `shared/src/schemas/mcp.ts` — Zod schemas for MCP tool inputs and outputs:
   - `TailorContextInput` — task (string, required), projectId (string UUID, required), maxTokens (number, optional, default 4000), includeWebSearch (boolean, optional, default true).
   - `TailorContextOutput` — context (string), qualityScore (number, optional), sources (array of { title, type, relevance }).
4. Create `mcp-server/src/lib/apiClient.ts` — HTTP client class that wraps calls to the backend API:
   - Constructor accepts `baseUrl` and `apiKey` from environment variables (`AGENTTAILOR_API_URL`, `AGENTTAILOR_API_KEY`).
   - `tailorContext(input)` — POST to `/api/tailor` with task, projectId, maxTokens, includeWebSearch. Returns assembled context.
   - Error handling: map HTTP errors to human-readable MCP tool errors.
5. Create `mcp-server/src/tools/tailorContext.ts` — MCP tool definition:
   - Tool name: `tailor_context`.
   - Description: "Assemble optimal context from project docs and web search for a given task."
   - Input schema derived from `TailorContextInput` Zod schema.
   - Handler calls `apiClient.tailorContext()`, formats response as structured text with source attribution.
6. Create `mcp-server/src/index.ts` — MCP server entry point:
   - Initialize MCP server with name "agenttailor" and version from package.json.
   - Register the `tailor_context` tool.
   - Connect via stdio transport (standard for Claude Desktop/Code).
   - Log startup and tool registration to stderr (MCP convention).
7. Add `mcp-server` to root `package.json` workspaces array.
8. Document MCP server configuration in a comment block in `mcp-server/src/index.ts` showing the `claude_desktop_config.json` snippet users need.

## Files to Create/Modify
- `mcp-server/package.json` — Package definition with MCP SDK dependency
- `mcp-server/tsconfig.json` — TypeScript config extending base
- `mcp-server/src/index.ts` — MCP server entry point with stdio transport
- `mcp-server/src/tools/tailorContext.ts` — tailor_context tool implementation
- `mcp-server/src/lib/apiClient.ts` — HTTP client for backend API calls
- `shared/src/schemas/mcp.ts` — Zod schemas for MCP tool I/O
- `package.json` — Add mcp-server to workspaces (modify)

## Acceptance Criteria
- [ ] `npm install` from root installs mcp-server workspace dependencies
- [ ] `npm run build -w mcp-server` compiles without TypeScript errors
- [ ] MCP server starts via `node mcp-server/dist/index.js` and registers tools on stdio
- [ ] `tailor_context` tool accepts task (string) and projectId (string) parameters
- [ ] Tool calls the backend `/api/tailor` endpoint and returns formatted context
- [ ] Missing or invalid parameters return descriptive MCP tool errors
- [ ] API client reads `AGENTTAILOR_API_URL` and `AGENTTAILOR_API_KEY` from environment
- [ ] Claude Desktop can connect to the MCP server and invoke `tailor_context`
