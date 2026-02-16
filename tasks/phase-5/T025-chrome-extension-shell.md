# T025 — Chrome Extension Shell (Manifest V3)

| Field | Value |
|-------|-------|
| Phase | 5 — Browser Extension |
| Agent Type | extension-agent |
| Depends On | T001 |
| Blocks | T026, T027, T028, T030 |
| Branch | `feat/T025-chrome-extension-shell` |
| Status | PENDING |

## Objective
Set up the Chrome Extension project with Manifest V3 — service worker for background processing, content script entry points, side panel for context preview, popup for quick settings, and declared permissions for chatgpt.com and claude.ai. Include a build pipeline using Vite with CRXJS or webpack.

## Context
The browser extension is the primary delivery mechanism for AgentTailor's value. It runs alongside ChatGPT and Claude, detecting when the user needs context and injecting it seamlessly. Manifest V3 is required for Chrome Web Store publication and brings service workers (replacing background pages), declarative content scripts, and the side panel API. Getting the shell right — with proper permissions, build pipeline, and project structure — is critical for all subsequent extension tasks (T026-T030).

## Subtasks
1. Create `extension/package.json` — dependencies: React, TypeScript, webextension-polyfill. Dev dependencies: Vite (or webpack), CRXJS Vite plugin (or webpack-extension-reloader), @types/chrome.
2. Create `extension/tsconfig.json` — extends `tsconfig.base.json`, includes `@types/chrome`, targets ES2020 for service worker compatibility.
3. Create `extension/manifest.json` — Manifest V3 with: `permissions` (storage, sidePanel, activeTab), `host_permissions` (chatgpt.com, claude.ai), `content_scripts` (matches for both platforms), `background.service_worker`, `side_panel.default_path`, `action.default_popup`, icons.
4. Create `extension/src/background/serviceWorker.ts` — register side panel, listen for tab activation on supported domains, handle messages from content scripts and popup, manage extension lifecycle events.
5. Create `extension/src/popup/popup.html` — HTML shell for the popup with React mount point.
6. Create `extension/src/popup/Popup.tsx` — minimal popup component: AgentTailor logo, active project display, quick toggle (On/Off), link to open side panel.
7. Create `extension/src/sidepanel/sidepanel.html` — HTML shell for the side panel with React mount point.
8. Create `extension/src/sidepanel/SidePanel.tsx` — placeholder side panel component with "AgentTailor Side Panel" heading and connection status.
9. Create `extension/src/content/index.ts` — content script entry point that detects which platform the user is on (chatgpt.com or claude.ai), logs detection, and sends a message to the service worker.
10. Configure build pipeline — `extension/vite.config.ts` (or `webpack.config.js`) that builds all entry points (service worker, content scripts, popup, side panel), handles React JSX, TypeScript, and outputs to `extension/dist/`.
11. Add `extension:dev` and `extension:build` scripts to root `package.json`.

## Files to Create/Modify
- `extension/manifest.json` — Chrome Manifest V3 definition
- `extension/package.json` — Extension dependencies and scripts
- `extension/tsconfig.json` — TypeScript configuration for extension
- `extension/src/background/serviceWorker.ts` — Service worker with message handling
- `extension/src/popup/Popup.tsx` — Popup React component
- `extension/src/popup/popup.html` — Popup HTML shell
- `extension/src/sidepanel/SidePanel.tsx` — Side panel React component
- `extension/src/sidepanel/sidepanel.html` — Side panel HTML shell
- `extension/src/content/index.ts` — Content script entry with platform detection
- `extension/vite.config.ts` — Build configuration (or webpack.config.js)
- `package.json` — Modify: add extension:dev, extension:build scripts

## Acceptance Criteria
- [ ] `npm run extension:build` compiles all entry points to `extension/dist/` without errors
- [ ] Extension can be loaded as unpacked extension in Chrome via `chrome://extensions`
- [ ] Service worker activates and logs lifecycle events
- [ ] Popup opens on extension icon click and renders React component
- [ ] Side panel opens on supported domains and renders React component
- [ ] Content script loads on chatgpt.com and claude.ai pages
- [ ] Content script correctly identifies which platform it is running on
- [ ] Manifest declares correct permissions (storage, sidePanel, activeTab)
- [ ] Manifest declares host_permissions for chatgpt.com and claude.ai
- [ ] Message passing works between content script, service worker, and popup
- [ ] TypeScript compilation succeeds with strict mode
- [ ] Build output is optimized for production (minified, no source maps in prod)
