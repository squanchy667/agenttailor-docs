# Tech Stack

## Technology Choices

| Technology | Version | Purpose |
|-----------|---------|---------|
| React | ^18 | UI framework |
| TypeScript | ^5 | Type safety |
| Vite | ^5 | Build tool and dev server |
| Tailwind CSS | ^3 | Utility-first CSS |
| Express.js | ^4 | API server |
| Zod | ^3 | Runtime validation and schema definitions |
| React Query | ^5 | Server state management |
| Vitest | ^1 | Test runner |

## Rationale

| Choice | Why |
|--------|-----|
| React + Vite | Fast HMR, modern tooling, large ecosystem for complex UIs like the agent builder |
| Tailwind CSS | Rapid UI development, consistent design system, small bundle with purging |
| Express.js | Lightweight, flexible, well-suited for the config processing API |
| Zod | Schema-first approach — define once, validate everywhere, infer TypeScript types |
| Vitest | Vite-native, fast execution, compatible API with Jest |
