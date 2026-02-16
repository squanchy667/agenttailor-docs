# Data Flow

## Request/Response Flow

```
User → Agent Builder UI → API Request → Express Router → Service Layer → Response → UI Update
```

## Data Transformation

```
Internal Agent Format (Zod schema)
        │
        ├── ChatGPT Custom GPT config
        ├── ChatGPT Assistants API JSON
        ├── Claude Project config
        └── Claude API messages format
```

## State Management

- **Server state** — React Query for API data (agent configs, templates)
- **Client state** — React state / context for UI state (builder selections, form inputs)
- **Form state** — Controlled forms with Zod validation

## Error Handling

- Zod validation errors surfaced as user-friendly field messages
- API errors returned in standard `{ error: { code, message } }` format
- Optimistic updates with rollback on failure
