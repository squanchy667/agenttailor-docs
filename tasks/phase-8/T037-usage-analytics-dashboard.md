# T037 — Usage Analytics Dashboard

| Field | Value |
|-------|-------|
| Phase | 8 — Polish & Launch |
| Agent Type | frontend-agent |
| Depends On | T019, T036 |
| Blocks | T040 |
| Branch | `feat/T037-usage-analytics-dashboard` |
| Status | PENDING |

## Objective
Build an analytics page in the dashboard that visualizes tailoring sessions over time, average quality scores, most-used projects, document statistics, and API usage against plan limits.

## Context
Analytics close the feedback loop for users. Without visibility into how they're using AgentTailor, users can't optimize their workflow or understand the value they're getting. The analytics dashboard answers key questions: "Am I using this enough to justify the cost?", "Is my context quality improving over time?", "Which project benefits most from AgentTailor?", and "Am I approaching my plan limit?". This page is also critical for converting free users to paid plans by showing them their usage trajectory.

## Subtasks
1. Create `server/src/services/analyticsService.ts` — Analytics data aggregation:
   - `getSessionsOverTime(userId, days)` — Group sessions by day for the last N days. Return array of { date, count }.
   - `getQualityTrend(userId, days)` — Average quality scores by day. Return array of { date, avgScore, minScore, maxScore }.
   - `getProjectStats(userId)` — Per-project breakdown: { projectId, projectName, sessionCount, avgQuality, documentCount }.
   - `getPlanUsage(userId)` — Current period usage: { used, limit, planTier, periodStart, periodEnd, percentUsed }.
   - `getSummaryStats(userId)` — Totals: { totalSessions, totalDocuments, avgQualityAllTime, activeProjects }.
2. Create `server/src/routes/analytics.ts` — Analytics API endpoints:
   - `GET /api/analytics/sessions?days=30` — Sessions over time.
   - `GET /api/analytics/quality?days=30` — Quality score trend.
   - `GET /api/analytics/projects` — Per-project statistics.
   - `GET /api/analytics/usage` — Plan usage data.
   - `GET /api/analytics/summary` — Summary statistics.
   - All endpoints require authentication and are scoped to the requesting user.
3. Create `dashboard/src/hooks/useAnalytics.ts` — React Query hooks:
   - `useSessionsOverTime(days)` — Fetches and caches session time series.
   - `useQualityTrend(days)` — Fetches and caches quality trend data.
   - `useProjectStats()` — Fetches per-project stats.
   - `usePlanUsage()` — Fetches current plan usage.
   - `useSummaryStats()` — Fetches summary statistics.
   - All hooks use appropriate staleTime (5 minutes) and refetch on window focus.
4. Create `dashboard/src/components/analytics/UsageChart.tsx`:
   - Bar chart showing tailoring sessions per day for the selected time range.
   - Time range selector: 7d, 30d, 90d.
   - Hover tooltip showing exact count per day.
   - Use Recharts (or similar lightweight chart library).
5. Create `dashboard/src/components/analytics/QualityTrend.tsx`:
   - Line chart showing average quality score over time with min/max range band.
   - Color-coded: green (80+), yellow (60-79), red (below 60).
   - Time range selector matching UsageChart.
6. Create `dashboard/src/components/analytics/ProjectStats.tsx`:
   - Table or card grid showing per-project statistics.
   - Columns: project name, session count, average quality, document count.
   - Sortable by any column.
   - Link to individual project pages.
7. Create `dashboard/src/components/analytics/PlanUsage.tsx`:
   - Circular progress or gauge showing current usage vs. plan limit.
   - Display: "X of Y requests used (Z%)" with plan tier name.
   - Warning state when usage exceeds 80%.
   - Upgrade CTA when on free plan and approaching limit.
8. Create `dashboard/src/pages/AnalyticsPage.tsx`:
   - Layout: summary stats cards at top, usage chart and quality trend side-by-side, project stats below, plan usage in sidebar or bottom.
   - Loading skeletons while data fetches.
   - Empty state for new users with no data yet.
   - Add route to dashboard router.

## Files to Create/Modify
- `dashboard/src/pages/AnalyticsPage.tsx` — Main analytics page layout
- `dashboard/src/components/analytics/UsageChart.tsx` — Sessions over time chart
- `dashboard/src/components/analytics/QualityTrend.tsx` — Quality score trend chart
- `dashboard/src/components/analytics/ProjectStats.tsx` — Per-project breakdown
- `dashboard/src/components/analytics/PlanUsage.tsx` — Usage vs. plan limit gauge
- `dashboard/src/hooks/useAnalytics.ts` — React Query hooks for analytics data
- `server/src/routes/analytics.ts` — Analytics API endpoints
- `server/src/services/analyticsService.ts` — Analytics data aggregation service

## Acceptance Criteria
- [ ] Analytics page loads and displays all sections without errors
- [ ] Usage chart shows sessions per day with selectable time ranges (7d, 30d, 90d)
- [ ] Quality trend chart displays average scores with min/max range bands
- [ ] Quality trend color-codes scores (green 80+, yellow 60-79, red below 60)
- [ ] Project stats show per-project session count, average quality, and document count
- [ ] Plan usage displays current requests vs. plan limit with percentage
- [ ] Plan usage shows warning state when exceeding 80% of limit
- [ ] Summary stats cards show totals at the top of the page
- [ ] Data refreshes after new tailoring sessions are created
- [ ] Empty state displays helpful message for users with no data
