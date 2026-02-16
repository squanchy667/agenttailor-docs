# API Reference

## Base URL

```
http://localhost:3001/api
```

## Authentication

All endpoints (except health) require Clerk authentication via Bearer token.

## Endpoints

### Health Check
```
GET /api/health
→ { "status": "ok", "timestamp": "..." }
```

### Auth
```
POST /api/auth/register
POST /api/auth/login
GET  /api/auth/me
```

### Projects
```
GET    /api/projects              → Project[]
POST   /api/projects              → Project
GET    /api/projects/:id          → Project
PUT    /api/projects/:id          → Project
DELETE /api/projects/:id          → 204
```

### Documents
```
POST   /api/projects/:id/documents          → Document (triggers processing)
GET    /api/projects/:id/documents          → Document[]
DELETE /api/projects/:id/documents/:docId   → 204
GET    /api/projects/:id/documents/:docId/status → { status, progress }
```

### Tailoring (THE CORE)
```
POST /api/tailor
Body: { projectId, taskInput, targetPlatform: "gpt"|"claude", options? }
→ { tailoredContext, sources, tokenCount, qualityScore, suggestions }

POST /api/tailor/preview
Body: { projectId, taskInput, targetPlatform }
→ { tailoredContext, tokenCount } (lighter, faster)

GET /api/tailor/sessions
→ TailoringSession[] (history)
```

### Search
```
POST /api/search/docs
Body: { projectId, query, limit? }
→ { results: ScoredChunk[] }

POST /api/search/web
Body: { query, limit? }
→ { results: WebSearchResult[] }
```

### Settings
```
GET /api/settings → UserSettings
PUT /api/settings → UserSettings
```

### Analytics
```
GET /api/analytics → { sessions, qualityTrend, projectStats, planUsage }
```

## Error Responses

All errors follow this format:
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": {}
  }
}
```

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Missing or invalid auth token |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `VALIDATION_ERROR` | 400 | Invalid request body |
| `RATE_LIMITED` | 429 | Plan quota exceeded |
| `PROCESSING_ERROR` | 500 | Document processing failure |
| `INTERNAL_ERROR` | 500 | Server error |
