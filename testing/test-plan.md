# Test Plan

## Strategy

- Unit tests for core logic (intelligence engine modules, chunking, embedding, schemas)
- Integration tests for API endpoints (routes with mocked services)
- Component tests for key React dashboard components
- E2E tests for critical flows (document upload → process → tailor → inject)

## Coverage Goals

| Area | Target |
|------|--------|
| Shared schemas | 95% |
| Intelligence engine services | 90% |
| Server routes | 85% |
| Document processing pipeline | 80% |
| Dashboard components | 75% |
| Overall | 80% |

## Test Types

| Type | Tool | Location |
|------|------|----------|
| Unit | Vitest | `*.test.ts` alongside source |
| Integration | Vitest + Supertest | `server/src/routes/__tests__/` |
| Component | Vitest + Testing Library | `dashboard/src/components/__tests__/` |
| E2E | Playwright | `e2e/` |

## Running Tests

```bash
npm run test              # Run all tests
npm run test:server       # Server tests only
npm run test:dashboard    # Dashboard tests only
npm run test:watch        # Watch mode
npm run test:coverage     # With coverage report
```

## Key Test Scenarios

### Intelligence Engine
- Task Analyzer correctly classifies coding/writing/analysis/research tasks
- Relevance Scorer reranks chunks more accurately than raw cosine similarity
- Gap Detector triggers web search when coverage is below threshold
- Context Compressor respects token budget and applies hierarchy correctly
- Source Synthesizer resolves contradictions between sources
- Context Window Manager enforces per-model token limits

### Document Pipeline
- Text extraction handles all supported formats (PDF, DOCX, MD, TXT, code)
- Semantic chunking preserves heading context and respects overlap config
- Embeddings are generated and stored in vector DB correctly
- Search returns relevant results ranked by similarity

### API Endpoints
- Project CRUD operations with auth
- Document upload triggers processing pipeline
- /api/tailor returns assembled context with quality scores
- Rate limiting enforces plan-specific quotas
- Error responses follow standard format

### Extension
- Content scripts detect chat input on ChatGPT and Claude
- Side panel displays assembled context with sources
- Context injection works for both platforms
