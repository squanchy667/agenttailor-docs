# T029 — Context Injection for Both Platforms

| Field | Value |
|-------|-------|
| Phase | 5 — Browser Extension |
| Agent Type | extension-agent |
| Depends On | T026, T027, T028, T015 |
| Blocks | T040 |
| Branch | `feat/T029-context-injection` |
| Status | PENDING |

## Objective
Implement the core injection flow — detect that the user has composed a message, call the backend /api/tailor endpoint, show context preview in the side panel, and inject the tailored context into the conversation input on user approval. Handle both ChatGPT and Claude platforms through the unified platform abstraction.

## Context
This is the capstone task of the browser extension — it connects everything together. The content scripts (T026, T027) provide detection and injection primitives, the side panel (T028) provides the review UI, and the Tailor Orchestrator API (T015) provides the assembled context. T029 wires them all together into the complete user flow: type a message, AgentTailor detects it, assembles relevant context, shows a preview, and on approval, prepends the context to the message. This is the moment AgentTailor delivers its core value.

## Subtasks
1. Create `extension/src/background/tailorBridge.ts` — the orchestration service in the service worker that coordinates the full flow:
   - Listen for "user composing message" events from content scripts
   - Extract the user's task/message text from the content script
   - Determine active project from extension settings (T030)
   - Call `POST /api/tailor` with task text, project ID, and platform
   - Stream progress updates to the side panel
   - On completion, send assembled context to the side panel for preview
   - On user approval, send inject command to the content script
   - Handle errors at each step with user-friendly messages
2. Modify `extension/src/content/chatgpt/injector.ts` — implement the actual injection logic: receive the approved context from the service worker, prepend it to the user's current input (with clear delimiter like `---\n[Context provided by AgentTailor]\n...\n---\n`), trigger React synthetic events to update ChatGPT's state.
3. Modify `extension/src/content/claude/injector.ts` — implement the actual injection logic for Claude's ProseMirror editor: insert context as formatted text blocks, dispatch proper ProseMirror-compatible events, handle Claude's paragraph structure.
4. Create `extension/src/sidepanel/hooks/useInjection.ts` — hook that manages the injection approval flow: receives context preview from service worker, provides approve/reject actions, tracks injection status (pending, approved, injected, failed).
5. Implement auto-tailor mode — when enabled in settings, skip the preview/approval step and inject automatically. Show a brief notification in the side panel instead.
6. Implement injection formatting — wrap the context in a clear, AI-readable format that includes: a preamble explaining this is tailored context, the context body, source citations, and a delimiter before the user's actual message.
7. Add telemetry events: tailoring triggered, tailoring completed, injection approved, injection completed, injection time.

## Files to Create/Modify
- `extension/src/background/tailorBridge.ts` — Full-flow orchestration in service worker
- `extension/src/content/chatgpt/injector.ts` — Modify: implement actual ChatGPT injection
- `extension/src/content/claude/injector.ts` — Modify: implement actual Claude injection
- `extension/src/sidepanel/hooks/useInjection.ts` — Injection approval flow hook
- `extension/src/shared/contextFormatter.ts` — Format context for AI-readable injection

## Acceptance Criteria
- [ ] Service worker detects when user composes a message on ChatGPT or Claude
- [ ] Service worker calls /api/tailor with the user's message and active project
- [ ] Side panel shows tailoring progress in real-time
- [ ] Side panel displays assembled context preview on completion
- [ ] User can approve or reject context injection from the side panel
- [ ] Approved context is injected into ChatGPT's input with proper React event triggers
- [ ] Approved context is injected into Claude's ProseMirror editor with proper events
- [ ] Injected context is wrapped in a clear format with preamble, body, citations, and delimiter
- [ ] Auto-tailor mode skips preview and injects automatically when enabled
- [ ] Injection does not break ChatGPT or Claude's input state or send functionality
- [ ] Error at any step (API failure, injection failure) shows user-friendly message in side panel
- [ ] Full flow completes in under 5 seconds for a project with reasonable document count
- [ ] Service worker handles multiple rapid requests gracefully (debounce/queue)
