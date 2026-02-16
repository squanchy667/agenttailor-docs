# T038 — Rate Limiting and Plan Enforcement

| Field | Value |
|-------|-------|
| Phase | 8 — Polish & Launch |
| Agent Type | backend-agent |
| Depends On | T003 |
| Blocks | T040 |
| Branch | `feat/T038-rate-limiting-plan-enforcement` |
| Status | PENDING |

## Objective
Implement Redis-based rate limiting per user plan tier — Free (50 requests/day), Pro (500 requests/day), Team (2000 requests/day) — with proper 429 responses, retry-after headers, and plan enforcement middleware.

## Context
Rate limiting is essential before launch for two reasons: protecting infrastructure from abuse and enforcing the business model. Without it, a single free-tier user could make unlimited API calls, consuming expensive embedding and search resources. The tiered approach aligns usage limits with pricing — free users get enough to experience value, paid users get enough for daily professional use. Redis is the right choice for rate tracking because it's already in the stack (Bull queues), supports atomic increment operations, and handles TTL-based resets naturally.

## Subtasks
1. Create `shared/src/schemas/plan.ts` — Plan definitions:
   - `PlanTier` enum: `FREE`, `PRO`, `TEAM`.
   - `PlanLimits` config object mapping each tier to limits:
     - FREE: { dailyRequests: 50, maxProjects: 3, maxDocsPerProject: 20, maxDocSizeMb: 5 }.
     - PRO: { dailyRequests: 500, maxProjects: 20, maxDocsPerProject: 100, maxDocSizeMb: 25 }.
     - TEAM: { dailyRequests: 2000, maxProjects: 100, maxDocsPerProject: 500, maxDocSizeMb: 50 }.
   - `RateLimitInfo` schema: { limit, remaining, reset (UTC timestamp), retryAfter (seconds) }.
2. Create or modify `server/src/lib/redis.ts` — Redis client singleton:
   - Export a shared Redis client (ioredis) configured from `REDIS_URL` environment variable.
   - Add connection error handling and reconnection logic.
   - If Redis is unavailable, log a warning and allow requests through (fail-open for availability).
3. Create `server/src/middleware/rateLimiter.ts` — Redis-based rate limiter:
   - Key format: `ratelimit:{userId}:{YYYY-MM-DD}` — tracks daily request count per user.
   - Use Redis `INCR` + `EXPIRE` for atomic increment with TTL (expire at midnight UTC).
   - Accept `limit` parameter from plan enforcer.
   - On limit exceeded: return 429 with JSON body `{ error: "Rate limit exceeded", rateLimit: RateLimitInfo }`.
   - Set response headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`, `Retry-After`.
   - Always set rate limit headers on successful requests too (so clients can track usage).
4. Create `server/src/middleware/planEnforcer.ts` — Plan enforcement middleware:
   - Look up user's plan tier from database (or cache in Redis for 5 minutes).
   - Resolve the correct `PlanLimits` for the tier.
   - Call rateLimiter with the tier's daily request limit.
   - Also enforce structural limits: reject project creation if at max projects, reject document upload if at max docs per project or file size exceeds tier limit.
5. Modify `server/src/routes/tailor.ts` — Add `planEnforcer` middleware to the tailoring endpoint.
6. Modify `server/src/routes/projects.ts` — Add plan enforcement to project creation (max projects) and document upload (max docs, max size).
7. Write unit tests for rate limiter logic with mock Redis.

## Files to Create/Modify
- `server/src/middleware/rateLimiter.ts` — Redis-based rate limiting middleware
- `server/src/middleware/planEnforcer.ts` — Plan tier enforcement middleware
- `shared/src/schemas/plan.ts` — PlanTier enum and PlanLimits configuration
- `server/src/lib/redis.ts` — Redis client singleton (create or modify)
- `server/src/routes/tailor.ts` — Add rate limit middleware (modify)
- `server/src/routes/projects.ts` — Add plan enforcement (modify)

## Acceptance Criteria
- [ ] Free tier is limited to 50 requests per day, Pro to 500, Team to 2000
- [ ] Rate limit counter resets at midnight UTC each day
- [ ] 429 response includes JSON body with `error`, `limit`, `remaining`, `reset`, and `retryAfter`
- [ ] `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` headers are set on all responses
- [ ] `Retry-After` header is set on 429 responses with seconds until reset
- [ ] Plan enforcer caches user plan tier in Redis (5-minute TTL) to avoid repeated DB lookups
- [ ] Project creation is blocked when user reaches max projects for their tier
- [ ] Document upload is blocked when project reaches max docs or file exceeds size limit for tier
- [ ] Rate limiter fails open (allows requests through) if Redis is unavailable
- [ ] Rate limiting tracks counts accurately under concurrent requests (atomic Redis operations)
