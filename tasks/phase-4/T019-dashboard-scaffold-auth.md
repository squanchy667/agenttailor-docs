# T019 — React Dashboard Scaffold with Auth Flow

| Field | Value |
|-------|-------|
| Phase | 4 — Dashboard Frontend |
| Agent Type | frontend-agent |
| Depends On | T003 |
| Blocks | T020, T037 |
| Branch | `feat/T019-dashboard-scaffold-auth` |
| Status | PENDING |

## Objective
Set up the React dashboard application with Clerk authentication integration (login/signup pages, protected routes), React Router for navigation, and the base layout structure (sidebar navigation, header with user menu, and content area).

## Context
The dashboard is where users manage their projects, upload documents, monitor processing status, and review tailoring sessions. Authentication is the first gate — every dashboard feature requires a logged-in user. Clerk provides hosted auth with minimal setup, letting us focus on the product rather than auth plumbing. The layout established here becomes the shell that all subsequent dashboard pages (T021-T024) plug into.

## Subtasks
1. Configure `dashboard/src/main.tsx` — wrap the app in `ClerkProvider` with publishable key from env, render `App` component inside `StrictMode`.
2. Create `dashboard/src/lib/clerk.ts` — Clerk provider configuration, export auth helpers (`useAuth`, `useUser` wrappers if needed), configure appearance theme to match AgentTailor branding.
3. Create `dashboard/src/App.tsx` — set up React Router with route structure: `/login` (public), `/signup` (public), `/` redirects to `/projects`, and all other routes wrapped in `<SignedIn>` guard. Include a catch-all 404 route.
4. Create `dashboard/src/pages/LoginPage.tsx` — render Clerk's `<SignIn>` component with redirect to `/projects` on success, styled to match app branding.
5. Create `dashboard/src/pages/SignupPage.tsx` — render Clerk's `<SignUp>` component with redirect to `/projects` on success.
6. Create `dashboard/src/components/layout/AppLayout.tsx` — the authenticated shell layout with Sidebar on the left, Header on top, and `<Outlet>` for page content. Responsive: sidebar collapses to hamburger on mobile.
7. Create `dashboard/src/components/layout/Sidebar.tsx` — navigation links: Projects, Documents, Tailoring, Settings. Active state highlighting, collapse/expand toggle, AgentTailor logo at top.
8. Create `dashboard/src/components/layout/Header.tsx` — breadcrumb area, search bar (placeholder), notification bell (placeholder), user avatar dropdown (profile, settings, sign out via Clerk).
9. Set up `dashboard/tailwind.config.ts` — extend theme with AgentTailor color palette (primary, secondary, accent), configure content paths, add custom font.
10. Create `dashboard/src/index.css` — Tailwind directives, CSS custom properties for theme colors, base typography styles.
11. Add `VITE_CLERK_PUBLISHABLE_KEY` to `.env.example`.

## Files to Create/Modify
- `dashboard/src/main.tsx` — App entry point with ClerkProvider
- `dashboard/src/App.tsx` — Router with auth guards and route definitions
- `dashboard/src/pages/LoginPage.tsx` — Clerk SignIn component wrapper
- `dashboard/src/pages/SignupPage.tsx` — Clerk SignUp component wrapper
- `dashboard/src/components/layout/AppLayout.tsx` — Authenticated shell layout
- `dashboard/src/components/layout/Sidebar.tsx` — Navigation sidebar
- `dashboard/src/components/layout/Header.tsx` — Top header with user menu
- `dashboard/src/lib/clerk.ts` — Clerk configuration and auth helpers
- `dashboard/tailwind.config.ts` — Tailwind theme configuration
- `dashboard/src/index.css` — Base styles and Tailwind directives
- `.env.example` — Add VITE_CLERK_PUBLISHABLE_KEY

## Acceptance Criteria
- [ ] Dashboard starts with `npm run dev` and loads at `http://localhost:5173`
- [ ] Unauthenticated users are redirected to `/login`
- [ ] Login page renders Clerk's SignIn component and redirects to `/projects` on success
- [ ] Signup page renders Clerk's SignUp component and redirects to `/projects` on success
- [ ] Authenticated users see the AppLayout with sidebar, header, and content area
- [ ] Sidebar shows navigation links: Projects, Documents, Tailoring, Settings
- [ ] Active sidebar link is visually highlighted
- [ ] Header displays user avatar with dropdown containing Sign Out
- [ ] Sign Out logs user out via Clerk and redirects to `/login`
- [ ] Layout is responsive — sidebar collapses on screens narrower than 768px
- [ ] Tailwind custom theme colors are applied consistently
