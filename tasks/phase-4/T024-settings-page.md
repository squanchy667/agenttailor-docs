# T024 — Settings and Preferences Page

| Field | Value |
|-------|-------|
| Phase | 4 — Dashboard Frontend |
| Agent Type | frontend-agent |
| Depends On | T020 |
| Blocks | — |
| Branch | `feat/T024-settings-page` |
| Status | PENDING |

## Objective
Build the settings page with API key management, default project selection, preferred model configuration, context assembly preferences (max chunks, compression level, web search toggle), and notification settings.

## Context
Users need control over how AgentTailor operates. API key management lets them provide their own keys for search providers. Context preferences let them tune the balance between comprehensive context and token efficiency. These settings flow through to the backend and affect every tailoring session. The settings page is also where power users configure advanced behavior without touching config files.

## Subtasks
1. Define settings schemas in `shared/src/schemas/settings.ts` — `UserSettings` (defaultProjectId, preferredPlatform, theme), `ContextPreferences` (maxChunks, compressionLevel: "none" | "light" | "aggressive", webSearchEnabled, webSearchMaxResults, chunkWeightThreshold), `ApiKeys` (tavilyApiKey, braveSearchApiKey — stored encrypted), `NotificationPreferences` (emailOnProcessingComplete, emailOnProcessingError).
2. Create `server/src/routes/settings.ts` — `GET /api/settings` (retrieve user settings), `PUT /api/settings` (update), `POST /api/settings/api-keys` (store encrypted API keys), `DELETE /api/settings/api-keys/:provider` (remove key). Auth required on all routes.
3. Create `dashboard/src/pages/SettingsPage.tsx` — tabbed layout with sections: General, Context Preferences, API Keys, Notifications. Each section is a separate card/form. Save button per section with success/error toast feedback.
4. Create `dashboard/src/components/settings/ApiKeyManager.tsx` — manage API keys for Tavily and Brave Search. Each key shows masked value if set, with reveal toggle, edit, and delete. Input for adding new keys with validation (test key before saving via backend). Clear indication of which keys are required vs. optional.
5. Create `dashboard/src/components/settings/ContextPreferences.tsx` — sliders/inputs for: max chunks per assembly (5-50, default 20), compression level (radio: None / Light / Aggressive with descriptions), web search toggle with max results slider (1-10), chunk relevance threshold (0.1-0.9 slider). Preview of estimated token usage at current settings.
6. Create `dashboard/src/components/settings/NotificationSettings.tsx` — toggles for email notifications: processing complete, processing error, weekly usage summary. Email address from Clerk profile (read-only display).
7. Create `dashboard/src/hooks/useSettings.ts` — React Query hooks: `useSettings()` (fetch), `useUpdateSettings()` (mutation), `useApiKeys()` (fetch masked keys), `useSaveApiKey()`, `useDeleteApiKey()`.

## Files to Create/Modify
- `shared/src/schemas/settings.ts` — UserSettings, ContextPreferences, ApiKeys, NotificationPreferences schemas
- `server/src/routes/settings.ts` — Settings CRUD API endpoints
- `dashboard/src/pages/SettingsPage.tsx` — Settings page with tabbed sections
- `dashboard/src/components/settings/ApiKeyManager.tsx` — API key management UI
- `dashboard/src/components/settings/ContextPreferences.tsx` — Context tuning controls
- `dashboard/src/components/settings/NotificationSettings.tsx` — Notification toggles
- `dashboard/src/hooks/useSettings.ts` — React Query hooks for settings

## Acceptance Criteria
- [ ] Settings page loads with current user settings pre-filled
- [ ] General section allows selecting default project and preferred platform
- [ ] Context preferences section has working sliders for max chunks and relevance threshold
- [ ] Compression level radio buttons show descriptive text for each option
- [ ] Web search toggle enables/disables the max results slider
- [ ] API key manager displays masked keys (e.g., `tvly-****...****ab3f`) for set keys
- [ ] Adding a new API key validates the key via backend before saving
- [ ] Deleting an API key shows confirmation and removes it
- [ ] Notification toggles save correctly and reflect current state
- [ ] Each settings section saves independently with success/error toast
- [ ] `GET /api/settings` returns user settings with API keys masked
- [ ] `PUT /api/settings` validates input with Zod and updates settings
- [ ] `POST /api/settings/api-keys` encrypts key before storing
- [ ] Settings changes persist across page reloads
