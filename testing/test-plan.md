# Test Plan

## Strategy

- Unit tests for business logic (export engine, schema validation, data transformations)
- Component tests for React UI components (rendering, user interactions)
- Integration tests for API endpoints (request/response validation)
- End-to-end tests for critical user flows (create agent → configure → export)

## Coverage Goals

| Area | Target |
|------|--------|
| Shared schemas/types | 95% |
| Server services | 90% |
| Server routes | 85% |
| React components | 80% |
| Overall | 85% |

## Test Types

| Type | Tool | Location |
|------|------|----------|
| Unit | Vitest | `*.test.ts` alongside source |
| Component | Vitest + Testing Library | `*.test.tsx` alongside source |
| Integration | Vitest + Supertest | `server/src/**/*.test.ts` |
| E2E | Playwright | `e2e/` |

## Running Tests

```bash
npm run test           # Run all tests
npm run test:watch     # Watch mode
npm run test:coverage  # With coverage report
```
