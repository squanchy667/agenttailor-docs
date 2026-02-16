# Coding Standards

## File Naming

- Components: `PascalCase.tsx` (e.g., `ContextViewer.tsx`)
- Hooks: `camelCase.ts` with `use` prefix (e.g., `useTailoring.ts`)
- Services: `camelCase.ts` (e.g., `tailorOrchestrator.ts`)
- Types/schemas: `camelCase.ts` (e.g., `tailor.ts`)
- Routes: `camelCase.ts` (e.g., `projects.ts`)
- Tests: `*.test.ts` or `*.test.tsx` alongside source files

## Code Style

- Strict TypeScript — no `any`, explicit return types on exports
- Functional components with named exports
- Zod schemas as source of truth for types (`z.infer<typeof Schema>`)
- Prefer `const` over `let`, never use `var`
- Destructure props and function parameters

## Import Conventions

- External packages first, then internal modules, then relative imports
- Use path aliases: `@server/`, `@dashboard/`, `@shared/`, `@extension/`
- Named exports preferred over default exports

## Error Handling

- Validate inputs at API boundaries with Zod
- Use typed error responses: `{ error: { code, message, details? } }`
- Document processing errors: update status in DB, push WebSocket notification
- Tailoring errors: return partial results with degraded quality score
- Never swallow errors silently — log or surface them

## Testing

- Co-locate test files with source (`service.test.ts` next to `service.ts`)
- Use `describe` / `it` blocks with clear test names
- Test behavior, not implementation details
- Mock external dependencies (OpenAI, Tavily, vector stores)
- Integration tests use supertest for API endpoints
