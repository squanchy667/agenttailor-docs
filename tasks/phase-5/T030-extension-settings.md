# T030 — Extension Settings and Project Selection

| Field | Value |
|-------|-------|
| Phase | 5 — Browser Extension |
| Agent Type | extension-agent |
| Depends On | T025 |
| Blocks | — |
| Branch | `feat/T030-extension-settings` |
| Status | PENDING |

## Objective
Build the extension popup and settings interface for selecting the active project, configuring the API endpoint, managing login state, and providing quick toggles for auto-tailor and web search features.

## Context
The extension popup is the quick-access control panel for AgentTailor. Users need to switch between projects (e.g., switching from working on "API Docs" to "Frontend Guide"), toggle features on/off, and verify their connection status without opening the full dashboard. Project selection is critical because it determines which document collection the tailoring engine searches. Settings are persisted in Chrome's extension storage and synced across devices.

## Subtasks
1. Create `extension/src/background/storage.ts` — typed wrapper around `chrome.storage.sync` and `chrome.storage.local`. Manages: `activeProjectId` (string), `apiEndpoint` (string, default: production URL), `authToken` (string, stored in local only — not synced), `autoTailor` (boolean, default: false), `webSearchEnabled` (boolean, default: true), `theme` (light/dark/system). Provides `getSettings()`, `updateSettings(partial)`, and `onSettingsChanged(callback)` methods.
2. Create `extension/src/popup/Popup.tsx` — redesign the placeholder popup into a full settings interface:
   - Connection status indicator (green dot = connected, red = disconnected)
   - Active project selector (dropdown populated from API)
   - Auto-tailor toggle with explanation tooltip
   - Web search toggle
   - "Open Dashboard" link (opens dashboard in new tab)
   - "Open Side Panel" button
   - Sign in / Sign out button
   - Version number footer
3. Create `extension/src/popup/components/ProjectSelector.tsx` — dropdown that fetches user's projects from `GET /api/projects`, displays them with name and document count, remembers last selection. Shows loading state while fetching, error state if API unreachable. "No projects yet — create one in the dashboard" empty state.
4. Create `extension/src/popup/components/QuickSettings.tsx` — toggle switches for: auto-tailor (on/off with description: "Automatically inject context without preview"), web search (on/off with description: "Include web search results in context"). Each toggle saves immediately to extension storage.
5. Create `extension/src/popup/hooks/useExtensionSettings.ts` — hook that reads/writes extension settings from Chrome storage, provides reactive state that updates when storage changes (from other extension views like the side panel).
6. Implement API endpoint configuration — for development, allow users to point the extension at `http://localhost:4000` instead of production. Accessible via a "Developer Settings" collapsible section in the popup.
7. Implement login flow — popup opens a new tab to the dashboard login page, waits for auth token to be passed back via `chrome.runtime.sendMessage` from the dashboard, stores the token in extension storage.

## Files to Create/Modify
- `extension/src/popup/Popup.tsx` — Full popup UI with project selector and settings
- `extension/src/popup/components/ProjectSelector.tsx` — Project dropdown selector
- `extension/src/popup/components/QuickSettings.tsx` — Auto-tailor and web search toggles
- `extension/src/background/storage.ts` — Extension storage wrapper with typed API
- `extension/src/popup/hooks/useExtensionSettings.ts` — Reactive settings hook

## Acceptance Criteria
- [ ] Popup opens on extension icon click and renders within 200ms
- [ ] Connection status shows green when API is reachable, red when disconnected
- [ ] Project selector loads user's projects from the API
- [ ] Selecting a project saves it to extension storage and updates the active project
- [ ] Project selector shows document count for each project
- [ ] Auto-tailor toggle saves immediately and affects injection behavior (T029)
- [ ] Web search toggle saves immediately and is passed to the /api/tailor calls
- [ ] "Open Dashboard" opens the dashboard in a new browser tab
- [ ] "Open Side Panel" opens the extension side panel on the current tab
- [ ] Sign in flow opens dashboard login and retrieves auth token
- [ ] Sign out clears the stored auth token and shows signed-out state
- [ ] Settings persist across browser restarts via chrome.storage.sync
- [ ] Auth token is stored in chrome.storage.local (not synced across devices)
- [ ] Developer settings allow changing the API endpoint to localhost
- [ ] Empty state shows when user has no projects
