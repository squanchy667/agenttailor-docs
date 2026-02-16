# T021 — Project Management Pages

| Field | Value |
|-------|-------|
| Phase | 4 — Dashboard Frontend |
| Agent Type | frontend-agent |
| Depends On | T020, T004 |
| Blocks | — |
| Branch | `feat/T021-project-management-pages` |
| Status | PENDING |

## Objective
Build the project list page (card grid with create, edit, and delete capabilities) and the project detail page (project overview, document list, and recent tailoring sessions).

## Context
Projects are the top-level organizational unit in AgentTailor. Users create a project for each codebase, product, or knowledge domain they want to tailor AI context for. The project management pages are the first thing users see after login and the primary navigation hub. The project list connects to the Projects CRUD API (T004), making this the first full frontend-to-backend integration in the dashboard.

## Subtasks
1. Create `dashboard/src/hooks/useProjects.ts` — React Query hooks: `useProjects()` (list all), `useProject(id)` (single), `useCreateProject()`, `useUpdateProject()`, `useDeleteProject()`. All use the typed API client from T020, with optimistic updates for create/delete.
2. Create `dashboard/src/pages/ProjectsPage.tsx` — grid layout of ProjectCards with a "New Project" button. Search/filter bar at top. Empty state when no projects exist (illustration + CTA). Loading state shows skeleton cards.
3. Create `dashboard/src/components/projects/ProjectCard.tsx` — displays project name, description (truncated), document count, last tailoring date, status badge. Click navigates to project detail. Three-dot menu with Edit and Delete options.
4. Create `dashboard/src/components/projects/ProjectForm.tsx` — form for creating/editing projects. Fields: name (required), description (textarea), default platform (ChatGPT/Claude dropdown), tags (multi-select/free text). Used in a Modal for both create and edit flows.
5. Create `dashboard/src/pages/ProjectDetailPage.tsx` — project header (name, description, edit button), three tabs or sections: Overview (stats cards — doc count, chunk count, session count, last active), Documents (list with upload button linking to T022), Recent Sessions (list linking to T023).
6. Add routes in `dashboard/src/App.tsx`: `/projects` (list), `/projects/:projectId` (detail).
7. Implement delete confirmation dialog using the Modal component.

## Files to Create/Modify
- `dashboard/src/pages/ProjectsPage.tsx` — Project list page with grid layout
- `dashboard/src/pages/ProjectDetailPage.tsx` — Project detail page with tabs
- `dashboard/src/components/projects/ProjectCard.tsx` — Individual project card
- `dashboard/src/components/projects/ProjectForm.tsx` — Create/edit project form
- `dashboard/src/hooks/useProjects.ts` — React Query hooks for project CRUD
- `dashboard/src/App.tsx` — Modify: add project routes

## Acceptance Criteria
- [ ] Project list page displays all user projects in a responsive card grid
- [ ] "New Project" button opens a modal with the project creation form
- [ ] Project creation form validates name (required, min 2 chars) and submits to API
- [ ] Newly created project appears in the list without full page reload (optimistic update)
- [ ] Project card shows name, description preview, document count, and last active date
- [ ] Clicking a project card navigates to `/projects/:projectId`
- [ ] Project detail page displays project name, description, and stats
- [ ] Project detail page shows document list and recent sessions sections
- [ ] Edit project opens pre-filled form modal and saves changes
- [ ] Delete project shows confirmation dialog and removes project on confirm
- [ ] Empty state renders when user has no projects (with illustration and "Create your first project" CTA)
- [ ] Loading state shows skeleton cards while data is fetching
- [ ] Error state displays ErrorBanner with retry option
