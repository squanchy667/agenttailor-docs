# T028 — Side Panel UI with Context Preview

| Field | Value |
|-------|-------|
| Phase | 5 — Browser Extension |
| Agent Type | extension-agent |
| Depends On | T025 |
| Blocks | T029 |
| Branch | `feat/T028-side-panel-ui` |
| Status | PENDING |

## Objective
Build the extension side panel that shows "Tailoring..." status during context assembly, a preview of the assembled context, a source list with relevance scores, and controls for the user to review and edit the context before injection into the AI conversation.

## Context
The side panel is the user's window into what AgentTailor is doing. When the extension detects a task and calls the backend, the side panel shows real-time status. Once context is assembled, the user can review each source, see why it was included, edit the context if needed, and approve injection. This human-in-the-loop design builds trust and gives users control. The side panel communicates with the service worker to get tailoring results and with content scripts to trigger injection.

## Subtasks
1. Create `extension/src/sidepanel/SidePanel.tsx` — main panel component with states: Idle (no active tailoring, shows last session or welcome), Tailoring (loading animation with status text: "Detecting task...", "Searching documents...", "Assembling context..."), Preview (assembled context ready for review), Injected (context was sent to the AI, showing confirmation).
2. Create `extension/src/sidepanel/components/StatusIndicator.tsx` — visual status component: animated dots/spinner during tailoring, green check on success, red X on error. Shows elapsed time during tailoring.
3. Create `extension/src/sidepanel/components/ContextPreview.tsx` — displays the assembled context in a scrollable, syntax-highlighted view. Shows total token count. Sections are visually separated by source. Supports expand/collapse for long sections.
4. Create `extension/src/sidepanel/components/SourceList.tsx` — list of sources that contributed to the context. Each item shows: source name (doc title or web URL), type badge (Doc/Web), relevance score bar (0-100%), chunk count from this source. Sortable by relevance. Click expands to show the actual chunk text.
5. Create `extension/src/sidepanel/components/EditableContext.tsx` — textarea/editor that lets the user modify the assembled context before injection. "Reset to original" button. Character and token count that updates as user edits. Warning if context exceeds platform token limits.
6. Create `extension/src/sidepanel/hooks/useTailoring.ts` — hook that communicates with the service worker to: trigger tailoring, receive progress updates, get assembled context, approve/reject injection. Uses `chrome.runtime.sendMessage` and `chrome.runtime.onMessage`.
7. Add action buttons at the bottom of the panel: "Inject Context" (primary, triggers injection into the AI chat), "Copy to Clipboard" (secondary), "Dismiss" (cancel without injecting).

## Files to Create/Modify
- `extension/src/sidepanel/SidePanel.tsx` — Main side panel component with state management
- `extension/src/sidepanel/components/StatusIndicator.tsx` — Tailoring status display
- `extension/src/sidepanel/components/ContextPreview.tsx` — Assembled context viewer
- `extension/src/sidepanel/components/SourceList.tsx` — Source list with relevance scores
- `extension/src/sidepanel/components/EditableContext.tsx` — Editable context before injection
- `extension/src/sidepanel/hooks/useTailoring.ts` — Service worker communication hook

## Acceptance Criteria
- [ ] Side panel displays Idle state with welcome message when no tailoring is active
- [ ] Side panel transitions to Tailoring state with animated status indicator on trigger
- [ ] Status indicator shows elapsed time and descriptive progress text
- [ ] Assembled context is displayed in a readable, scrollable view after tailoring completes
- [ ] Token count is displayed and updates when context is edited
- [ ] Source list shows all contributing sources with type badge and relevance score
- [ ] Sources are sortable by relevance score
- [ ] Clicking a source expands to show the actual chunk text
- [ ] Editable context area allows user modifications with "Reset to original" option
- [ ] Warning displays when context exceeds platform-specific token limits
- [ ] "Inject Context" button sends the (possibly edited) context to the content script
- [ ] "Copy to Clipboard" button copies the assembled context
- [ ] "Dismiss" button returns the panel to Idle state without injecting
- [ ] Side panel communicates with service worker via Chrome messaging API
