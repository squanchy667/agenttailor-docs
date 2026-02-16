# T026 — ChatGPT Content Script and Detection

| Field | Value |
|-------|-------|
| Phase | 5 — Browser Extension |
| Agent Type | extension-agent |
| Depends On | T025 |
| Blocks | T029 |
| Branch | `feat/T026-chatgpt-content-script` |
| Status | PENDING |

## Objective
Build the content script that detects when the user is on chatgpt.com, identifies the chat input area, detects when the user starts typing a new message or creates a new conversation, and exposes injection points for inserting tailored context into the chat input.

## Context
ChatGPT's web interface is a React single-page application with dynamic DOM updates. The content script must observe DOM mutations to detect navigation between conversations, identify the current input textarea, and provide reliable injection methods. This is the "eyes and hands" of AgentTailor on the ChatGPT side — detection determines when to trigger tailoring, and injection is how the assembled context reaches the AI. ChatGPT's DOM structure changes periodically with updates, so the selectors need to be resilient with fallback strategies.

## Subtasks
1. Create `extension/src/content/chatgpt/detector.ts` — functions to detect: (a) whether the user is in a conversation or on the home page, (b) the current conversation ID from the URL, (c) the chat input element (textarea or contenteditable div — ChatGPT has used both), (d) the send button element. Use multiple selector strategies with fallbacks.
2. Create `extension/src/content/chatgpt/domObserver.ts` — `MutationObserver` that watches for: (a) SPA route changes (conversation switches), (b) chat input becoming available (new conversation or page load), (c) user typing in the input (input/change events), (d) message submission (detect new message appearing in conversation). Emits typed events that the injector and bridge can listen to.
3. Create `extension/src/content/chatgpt/injector.ts` — methods to inject text into ChatGPT's chat input: (a) `injectContext(text: string)` — prepends tailored context before the user's message, (b) `getInputContent()` — reads current input value, (c) `setInputContent(text: string)` — replaces input content. Must handle both textarea and contenteditable elements. Trigger React's synthetic events so ChatGPT's state updates correctly.
4. Create `extension/src/content/chatgpt/index.ts` — orchestration: initialize detector, start DOM observer, expose messaging interface to the service worker via `chrome.runtime.onMessage`. Report platform status: `{ platform: "chatgpt", ready: boolean, conversationId: string | null, inputDetected: boolean }`.
5. Handle edge cases: ChatGPT loading states, network errors showing error pages, model selector modal blocking input, conversation loading skeleton.

## Files to Create/Modify
- `extension/src/content/chatgpt/detector.ts` — Page state and element detection
- `extension/src/content/chatgpt/domObserver.ts` — MutationObserver for SPA changes
- `extension/src/content/chatgpt/injector.ts` — Methods to inject text into chat input
- `extension/src/content/chatgpt/index.ts` — Orchestration and messaging interface

## Acceptance Criteria
- [ ] Detector identifies whether user is on ChatGPT home page vs. active conversation
- [ ] Detector extracts conversation ID from URL path
- [ ] Detector locates the chat input element using multiple selector strategies
- [ ] Detector locates the send button element
- [ ] DOM observer detects SPA navigation between conversations
- [ ] DOM observer detects when chat input becomes available after page load
- [ ] DOM observer fires event when user starts typing in the input
- [ ] Injector can prepend context text before the user's message in the input
- [ ] Injector triggers React synthetic events so ChatGPT's internal state updates
- [ ] Injector works with both textarea and contenteditable input variants
- [ ] Content script reports platform status to service worker via messaging
- [ ] Content script handles ChatGPT loading/error states gracefully without throwing
- [ ] Selector strategies include fallbacks for DOM structure changes
