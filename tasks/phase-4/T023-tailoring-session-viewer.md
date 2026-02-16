# T023 — Tailoring Session Viewer

| Field | Value |
|-------|-------|
| Phase | 4 — Dashboard Frontend |
| Agent Type | frontend-agent |
| Depends On | T020, T015 |
| Blocks | — |
| Branch | `feat/T023-tailoring-session-viewer` |
| Status | PENDING |

## Objective
Build the tailoring session viewer page that displays assembled context, explains why each piece was included (relevance score, source document/URL), and lets users manually try a tailoring request by entering a task description and selecting a target platform.

## Context
The session viewer is where users understand and debug what AgentTailor does. It answers the question: "What context did AgentTailor assemble for my task, and why?" This transparency is essential for user trust and for iterating on document collections. The interactive tailoring form also serves as a playground for users to test their setup before using the browser extension. This connects to the Tailor Orchestrator API (T015) and citation tracking (T018).

## Subtasks
1. Create `dashboard/src/pages/TailoringPage.tsx` — main page with two sections: the interactive tailoring form (top) and session history list (bottom). Route: `/tailoring`.
2. Create `dashboard/src/components/tailoring/TailorForm.tsx` — form with: project selector (dropdown of user's projects), task description (textarea, "What are you working on?"), target platform (ChatGPT / Claude toggle), web search toggle, and "Tailor Context" submit button. Shows loading state during API call.
3. Create `dashboard/src/components/tailoring/ContextViewer.tsx` — displays the assembled context output. Renders the full tailored text with inline annotations: each chunk is visually delineated with a colored left-border, hovering shows source metadata. Includes a "Copy to Clipboard" button for the full context and a character/token count display.
4. Create `dashboard/src/components/tailoring/SourceCard.tsx` — individual source card showing: source title (document name or web URL), relevance score (0-100% with color bar), source type badge (Document / Web), page/section info for docs, search query for web results. Expandable to show the full chunk text.
5. Create `dashboard/src/components/tailoring/SessionHistory.tsx` — list of past tailoring sessions with: timestamp, task description preview, project name, platform, chunk count, and a "View" link that loads the session details into the viewer.
6. Create `dashboard/src/hooks/useTailoring.ts` — React Query hooks: `useTailorContext()` (mutation that calls POST /api/tailor), `useSessions(projectId?)` (list sessions), `useSession(sessionId)` (single session details with context and citations).
7. Add route `/tailoring` and `/tailoring/:sessionId` to `dashboard/src/App.tsx`.

## Files to Create/Modify
- `dashboard/src/pages/TailoringPage.tsx` — Main tailoring page with form and history
- `dashboard/src/components/tailoring/TailorForm.tsx` — Interactive tailoring request form
- `dashboard/src/components/tailoring/ContextViewer.tsx` — Assembled context display with annotations
- `dashboard/src/components/tailoring/SourceCard.tsx` — Individual source card with relevance
- `dashboard/src/components/tailoring/SessionHistory.tsx` — Past session list
- `dashboard/src/hooks/useTailoring.ts` — React Query hooks for tailoring API
- `dashboard/src/App.tsx` — Modify: add tailoring routes

## Acceptance Criteria
- [ ] Tailoring form allows selecting a project, entering task description, and choosing platform
- [ ] Submitting the form calls POST /api/tailor and shows loading state
- [ ] Assembled context is displayed in ContextViewer with visual chunk boundaries
- [ ] Each chunk shows source attribution on hover (document name or web URL)
- [ ] Source cards display relevance score as percentage with color-coded bar
- [ ] Source cards distinguish between Document and Web sources with badges
- [ ] "Copy to Clipboard" button copies the full assembled context
- [ ] Token/character count is displayed for the assembled context
- [ ] Session history lists past sessions with timestamp, task preview, and project
- [ ] Clicking a session in history loads its details into the viewer
- [ ] Empty state shows when no sessions exist (with guidance to try the form)
- [ ] Web search toggle in the form controls whether web results are included
- [ ] Error state from failed tailoring request shows clear error message
