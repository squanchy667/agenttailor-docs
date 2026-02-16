# API Reference

## Base URL

```
http://localhost:3001/api
```

## Authentication

<!-- Auth method will be defined during planning -->

_Awaiting architecture planning._

## Endpoints

### Health Check

```
GET /api/health
```

Response:
```json
{ "status": "ok", "timestamp": "..." }
```

<!-- API endpoints will be defined during planning -->

_Run `/plan-project AgentTailor` to generate endpoint specifications._

## Error Responses

All errors follow this format:
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message"
  }
}
```

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Missing or invalid auth token |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `VALIDATION_ERROR` | 400 | Invalid request body |
| `INTERNAL_ERROR` | 500 | Server error |
