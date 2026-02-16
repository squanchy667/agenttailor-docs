# Express Routing Patterns

## Route Structure

```
server/src/routes/
├── index.ts          # Route registration
├── agents.ts         # /api/agents/*
├── templates.ts      # /api/templates/*
└── exports.ts        # /api/exports/*
```

## Route Template

```typescript
import { Router } from 'express';
import type { Request, Response } from 'express';

const router = Router();

// GET /api/resource
router.get('/', async (req: Request, res: Response) => {
  // Implementation
});

// GET /api/resource/:id
router.get('/:id', async (req: Request, res: Response) => {
  const id = req.params.id as string;
  // Implementation
});

// POST /api/resource
router.post('/', async (req: Request, res: Response) => {
  const body = RequestSchema.parse(req.body);
  // Implementation
});

export { router as resourceRouter };
```

## Middleware Stack

- `express.json()` — Parse JSON bodies
- `cors()` — Cross-origin requests
- `validateRequest(schema)` — Zod schema validation middleware
- Error handler — Catches Zod errors and returns standard error format

## Error Handling

```typescript
router.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  if (err instanceof z.ZodError) {
    return res.status(400).json({ error: { code: 'VALIDATION_ERROR', details: err.errors } });
  }
  res.status(500).json({ error: { code: 'INTERNAL_ERROR', message: 'Internal server error' } });
});
```
