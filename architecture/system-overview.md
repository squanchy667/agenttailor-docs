# System Overview

## Architecture

```
┌─────────────────────────────────────────┐
│              Client (React SPA)          │
│  ┌──────────┐ ┌──────────┐ ┌─────────┐ │
│  │  Agent    │ │  Tool    │ │ Template│ │
│  │  Builder  │ │  Config  │ │ Library │ │
│  └────┬─────┘ └────┬─────┘ └────┬────┘ │
│       └─────────────┼────────────┘      │
│                     │                    │
└─────────────────────┼────────────────────┘
                      │ REST API
┌─────────────────────┼────────────────────┐
│              Server (Express)            │
│  ┌──────────┐ ┌──────────┐ ┌─────────┐ │
│  │  Agent    │ │  Export   │ │ Template│ │
│  │  Service  │ │  Engine   │ │ Service │ │
│  └──────────┘ └──────────┘ └─────────┘ │
└──────────────────────────────────────────┘
```

## Components

<!-- Component descriptions will be detailed during /plan-project -->

- **Agent Builder** — Visual editor for composing agent configurations
- **Tool Configurator** — Schema-driven tool/function definition editor
- **Export Engine** — Transforms internal agent format into platform-specific configs
- **Template Library** — Pre-built agent templates with categorization

## Technology Choices

| Choice | Rationale |
|--------|-----------|
| React + Vite | Fast HMR, modern tooling, component ecosystem |
| Tailwind CSS | Utility-first approach, rapid UI development |
| Express.js | Lightweight, flexible API server |
| Zod | Runtime validation with TypeScript type inference |
| Vitest | Fast, Vite-native testing framework |

## Deployment

<!-- Deployment topology will be detailed during architecture planning -->

_Awaiting architecture planning._
