# T003 — Auth Integration with Clerk

| Field | Value |
|-------|-------|
| Phase | 1 — Foundation |
| Agent Type | backend-agent |
| Depends On | T002 |
| Blocks | T004, T019, T038 |
| Branch | `feat/T003-clerk-auth-integration` |
| Status | PENDING |

## Objective
Integrate Clerk for user authentication — signup, login, session management. Add auth middleware for protected API routes and sync Clerk users to the local database.

## Context
Every API endpoint beyond health checks needs authentication. Clerk provides managed auth with webhooks for user lifecycle events, JWTs for session verification, and SDKs for both server and client. The middleware must extract the authenticated user from Clerk tokens and attach the local database user to the request context.

## Subtasks
1. Install Clerk server SDK in `server/` — `@clerk/express`.
2. Create `server/src/middleware/auth.ts` with two middlewares:
   - `requireAuth` — Verifies Clerk JWT from Authorization header, rejects 401 if invalid. Attaches `req.auth` with Clerk session data.
   - `attachUser` — After `requireAuth`, looks up or creates the local User record by `clerkId`, attaches `req.user` with the Prisma User object.
3. Create `server/src/routes/auth.ts` with endpoints:
   - `POST /api/auth/webhook` — Clerk webhook handler for `user.created`, `user.updated`, `user.deleted` events. Syncs user data to local DB. Verifies webhook signature.
   - `GET /api/auth/me` — Returns the authenticated user's profile (local DB record merged with Clerk data).
   - `POST /api/auth/sync` — Manual sync of Clerk user to local DB (useful for first login).
4. Update `server/src/routes/index.ts` — Central router that mounts auth routes and applies `requireAuth` + `attachUser` to all `/api/*` routes except webhooks and health.
5. Create `shared/src/schemas/user.ts` — Zod schemas for UserProfile, UserSettings, and API response shapes.
6. Extend the Express Request type in `server/src/types/express.d.ts` to include `auth` and `user` properties.
7. Add Clerk environment variables to `.env.example`: `CLERK_SECRET_KEY`, `CLERK_PUBLISHABLE_KEY`, `CLERK_WEBHOOK_SECRET`.

## Files to Create/Modify
- `server/src/middleware/auth.ts` — requireAuth and attachUser middlewares
- `server/src/routes/auth.ts` — Webhook handler, /me, /sync endpoints
- `server/src/routes/index.ts` — Central router with auth middleware applied
- `shared/src/schemas/user.ts` — Zod schemas for user-related data
- `server/src/types/express.d.ts` — Extended Express Request type
- `server/package.json` — Add @clerk/express dependency
- `.env.example` — Add Clerk environment variables

## Acceptance Criteria
- [ ] Unauthenticated requests to protected endpoints return 401
- [ ] Valid Clerk JWT in Authorization header passes `requireAuth` middleware
- [ ] `GET /api/auth/me` returns the authenticated user's profile
- [ ] Clerk `user.created` webhook creates a new User record in PostgreSQL
- [ ] Clerk `user.updated` webhook updates the corresponding User record
- [ ] Clerk `user.deleted` webhook soft-deletes the User record
- [ ] Webhook signature verification rejects invalid payloads
- [ ] `req.user` is available in all protected route handlers with full Prisma User type
- [ ] Zod schemas in shared package validate user data correctly
