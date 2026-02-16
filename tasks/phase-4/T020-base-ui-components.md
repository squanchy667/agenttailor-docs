# T020 — Base UI Components and Layout

| Field | Value |
|-------|-------|
| Phase | 4 — Dashboard Frontend |
| Agent Type | frontend-agent |
| Depends On | T019 |
| Blocks | T021, T022, T023, T024 |
| Branch | `feat/T020-base-ui-components` |
| Status | PENDING |

## Objective
Build a reusable UI component library for the dashboard — Button (with variants), Input, TextArea, Card, Modal, Spinner, Badge, EmptyState, ErrorBanner, Toast notification system, and Skeleton loaders. Also set up the API client and React Query for server state management.

## Context
Every dashboard page (T021-T024) needs consistent, well-tested UI primitives. Building these as a shared component library before the pages ensures visual consistency, reduces duplication, and speeds up page development. The API client and React Query setup establish the data-fetching pattern that all page-level hooks will follow.

## Subtasks
1. Create `dashboard/src/components/ui/Button.tsx` — variants: primary, secondary, outline, ghost, danger. Sizes: sm, md, lg. Support loading state (shows spinner), disabled state, icon slot (left/right). Use `cva` (class-variance-authority) or manual Tailwind class composition.
2. Create `dashboard/src/components/ui/Input.tsx` — text input with label, error message, helper text, prefix/suffix icons. Forward ref for form library compatibility.
3. Create `dashboard/src/components/ui/Card.tsx` — container with optional header, body, footer sections. Variants: default (shadow), outlined, flat.
4. Create `dashboard/src/components/ui/Modal.tsx` — dialog overlay with title, body, footer actions. Keyboard accessible (Escape to close, focus trap). Uses React Portal.
5. Create `dashboard/src/components/ui/Spinner.tsx` — animated loading spinner with size variants (sm, md, lg) and optional label.
6. Create `dashboard/src/components/ui/Badge.tsx` — status badges with color variants: default, success, warning, error, info. Used for document processing status, relevance indicators.
7. Create `dashboard/src/components/ui/Toast.tsx` — toast notification component with variants: success, error, warning, info. Auto-dismiss with configurable duration.
8. Create `dashboard/src/hooks/useToast.ts` — toast state management hook using React context. Provides `toast.success()`, `toast.error()`, `toast.warning()`, `toast.info()` methods. Manages toast queue with max 3 visible.
9. Create `dashboard/src/components/ui/Skeleton.tsx` — skeleton loading placeholders: SkeletonText, SkeletonCard, SkeletonAvatar, SkeletonTable. Pulse animation.
10. Create `dashboard/src/components/ui/index.ts` — barrel export for all UI components.
11. Create `dashboard/src/lib/api.ts` — typed fetch client that includes Clerk auth token in headers, handles JSON parsing, error responses (throws typed errors), and base URL configuration.
12. Create `dashboard/src/lib/queryClient.ts` — React Query client setup with default options (staleTime, retry logic, error handling), QueryClientProvider integration.

## Files to Create/Modify
- `dashboard/src/components/ui/Button.tsx` — Button component with variants
- `dashboard/src/components/ui/Input.tsx` — Input component with label and error
- `dashboard/src/components/ui/Card.tsx` — Card container component
- `dashboard/src/components/ui/Modal.tsx` — Modal dialog component
- `dashboard/src/components/ui/Spinner.tsx` — Loading spinner component
- `dashboard/src/components/ui/Badge.tsx` — Status badge component
- `dashboard/src/components/ui/Toast.tsx` — Toast notification component
- `dashboard/src/components/ui/Skeleton.tsx` — Skeleton loader components
- `dashboard/src/components/ui/index.ts` — Barrel export
- `dashboard/src/hooks/useToast.ts` — Toast state management hook
- `dashboard/src/lib/api.ts` — Typed API fetch client with auth
- `dashboard/src/lib/queryClient.ts` — React Query client configuration

## Acceptance Criteria
- [ ] Button renders all variants (primary, secondary, outline, ghost, danger) with correct styles
- [ ] Button loading state shows spinner and disables click
- [ ] Input displays label, handles error state with red border and error message
- [ ] Card renders with header, body, and footer sections
- [ ] Modal opens/closes, traps focus, closes on Escape key and overlay click
- [ ] Spinner renders in three sizes with smooth animation
- [ ] Badge displays correct color for each variant (success=green, error=red, etc.)
- [ ] Toast system shows notifications that auto-dismiss after configurable duration
- [ ] Toast queue limits visible toasts to 3, queuing additional ones
- [ ] Skeleton components render with pulse animation
- [ ] API client includes Clerk auth token in Authorization header
- [ ] API client throws typed errors for non-2xx responses
- [ ] React Query client is configured with sensible defaults (30s staleTime, 1 retry)
- [ ] All components are exported from the barrel `index.ts`
