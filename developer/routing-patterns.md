# Express Routing Patterns

## Route Structure

```
server/src/routes/
├── index.ts          # Route registration
├── auth.ts           # /api/auth/*
├── projects.ts       # /api/projects/*
├── documents.ts      # /api/projects/:id/documents/*
├── tailor.ts         # /api/tailor/*
├── search.ts         # /api/search/*
├── settings.ts       # /api/settings
├── analytics.ts      # /api/analytics
├── exports.ts        # /api/exports (if needed)
├── imports.ts        # /api/imports (if needed)
├── gptActions.ts     # /gpt/* (GPT Store Actions)
├── preview.ts        # /api/preview (if needed)
└── health.ts         # /api/health
```

## Route Template

```typescript
import { Router } from 'express';
import type { Request, Response } from 'express';
import { auth } from '../middleware/auth';
import { validateRequest } from '../middleware/validateRequest';

const router = Router();

router.get('/', auth, async (req: Request, res: Response) => {
  // Implementation
});

router.post('/', auth, validateRequest(CreateSchema, 'body'), async (req: Request, res: Response) => {
  const body = req.body; // Already validated by middleware
  // Implementation
});

export { router as resourceRouter };
```

## Middleware Stack

- `express.json()` — Parse JSON bodies
- `cors()` — Cross-origin requests (dashboard, extension)
- `auth` — Clerk authentication (via @clerk/express)
- `rateLimiter` — Redis-based rate limiting per user plan
- `validateRequest(schema)` — Zod schema validation middleware
- Error handler — Catches Zod errors and returns standard error format

## Error Handling

```typescript
router.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  if (err instanceof z.ZodError) {
    return res.status(400).json({
      error: { code: 'VALIDATION_ERROR', message: 'Invalid request', details: err.errors }
    });
  }
  res.status(500).json({
    error: { code: 'INTERNAL_ERROR', message: 'Internal server error' }
  });
});
```
