# T027 — Claude Content Script and Detection

| Field | Value |
|-------|-------|
| Phase | 5 — Browser Extension |
| Agent Type | extension-agent |
| Depends On | T025 |
| Blocks | T029 |
| Branch | `feat/T027-claude-content-script` |
| Status | PENDING |

## Objective
Build the content script for claude.ai that detects the chat input area, conversation state, and exposes injection points. Handle Claude's specific DOM structure (ProseMirror-based editor) and SPA routing.

## Context
Claude's web interface uses a different tech stack than ChatGPT — notably a ProseMirror-based rich text editor for input rather than a plain textarea. This means injection requires ProseMirror-aware methods to insert text without breaking the editor state. Claude's SPA routing and conversation structure also differ. This content script mirrors the ChatGPT one (T026) in purpose but diverges significantly in implementation details.

## Subtasks
1. Create `extension/src/content/claude/detector.ts` — functions to detect: (a) whether the user is in a conversation or on the home/new-chat page, (b) the current conversation ID from the URL, (c) the chat input element (ProseMirror editor div with `contenteditable="true"`), (d) the send button element, (e) whether the user is in a project context. Use resilient selectors with fallbacks.
2. Create `extension/src/content/claude/domObserver.ts` — `MutationObserver` that watches for: (a) SPA route changes (conversation switches, new chat), (b) ProseMirror editor becoming available, (c) user typing (input events on the ProseMirror editor), (d) message submission (detect Claude's response appearing). Emits typed events matching the same interface as the ChatGPT observer for consistency.
3. Create `extension/src/content/claude/injector.ts` — methods to inject text into Claude's ProseMirror editor: (a) `injectContext(text: string)` — insert tailored context into the editor, (b) `getInputContent()` — read current editor content as plain text, (c) `setInputContent(text: string)` — replace editor content. Must dispatch proper ProseMirror transactions or input events so Claude's frontend state stays in sync. Handle the editor's paragraph/block structure.
4. Create `extension/src/content/claude/index.ts` — orchestration: initialize detector, start DOM observer, expose messaging interface to service worker. Report platform status: `{ platform: "claude", ready: boolean, conversationId: string | null, inputDetected: boolean, projectContext: string | null }`.
5. Handle Claude-specific edge cases: "Thinking" indicator blocking input, conversation length limits, artifact panels, project context selection UI.

## Files to Create/Modify
- `extension/src/content/claude/detector.ts` — Page state and element detection for Claude
- `extension/src/content/claude/domObserver.ts` — MutationObserver for Claude's SPA
- `extension/src/content/claude/injector.ts` — ProseMirror-aware text injection
- `extension/src/content/claude/index.ts` — Orchestration and messaging interface

## Acceptance Criteria
- [ ] Detector identifies whether user is on Claude home page vs. active conversation
- [ ] Detector extracts conversation ID from Claude's URL structure
- [ ] Detector locates the ProseMirror editor element
- [ ] Detector locates the send button element
- [ ] Detector identifies project context when user is in a Claude project
- [ ] DOM observer detects SPA navigation between conversations on claude.ai
- [ ] DOM observer detects when the ProseMirror editor becomes available
- [ ] DOM observer fires event when user starts typing in the editor
- [ ] Injector inserts text into ProseMirror editor without breaking editor state
- [ ] Injector dispatches proper events so Claude's frontend state stays in sync
- [ ] `getInputContent()` correctly reads multi-line/multi-paragraph editor content
- [ ] Content script reports platform status to service worker via messaging
- [ ] Content script handles Claude's "Thinking" state and loading states gracefully
- [ ] Event interface is consistent with ChatGPT content script for code reuse in T029
