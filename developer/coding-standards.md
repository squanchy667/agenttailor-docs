# Coding Standards

## File Naming

- Components: `PascalCase.tsx` (e.g., `AgentBuilder.tsx`)
- Hooks: `camelCase.ts` with `use` prefix (e.g., `useAgentConfig.ts`)
- Utils/services: `camelCase.ts` (e.g., `exportEngine.ts`)
- Types/schemas: `camelCase.ts` (e.g., `agentSchema.ts`)
- Routes: `camelCase.ts` (e.g., `agents.ts`)
- Tests: `*.test.ts` or `*.test.tsx` alongside source files

## Code Style

- Strict TypeScript — no `any`, explicit return types on exports
- Functional components with named exports
- Zod schemas as source of truth for types (`z.infer<typeof Schema>`)
- Prefer `const` over `let`, never use `var`
- Destructure props and function parameters

## Import Conventions

- External packages first, then internal modules, then relative imports
- Use path aliases: `@client/`, `@server/`, `@shared/`
- Named exports preferred over default exports

## Error Handling

- Validate inputs at API boundaries with Zod
- Use typed error responses with consistent format
- Never swallow errors silently — log or surface them

## Testing

- Co-locate test files with source (`Component.test.tsx` next to `Component.tsx`)
- Use `describe` / `it` blocks with clear test names
- Test behavior, not implementation details
- Mock external dependencies, not internal modules
