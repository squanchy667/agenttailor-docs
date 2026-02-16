# Setup Guide

## Prerequisites

- Node.js >= 18
- npm >= 9

## Installation

```bash
git clone <repo-url> agenttailor
cd agenttailor
npm install
```

## Environment Variables

```bash
cp .env.example .env
```

| Variable | Description | Default |
|----------|-------------|---------|
| `PORT` | Server port | `3001` |
| `VITE_API_URL` | API base URL | `http://localhost:3001/api` |

## Running Locally

```bash
npm run dev          # Start both client and server
npm run dev:client   # Start client only
npm run dev:server   # Start server only
```

## Project Structure

```
agenttailor/
├── client/              # React frontend (Vite)
│   └── src/
│       ├── components/  # Reusable UI components
│       ├── pages/       # Route page components
│       ├── hooks/       # Custom React hooks
│       └── lib/         # Utilities, API client
├── server/              # Express backend
│   └── src/
│       ├── routes/      # API route handlers
│       ├── services/    # Business logic
│       └── middleware/  # Express middleware
├── shared/              # Shared types and Zod schemas
└── package.json         # Workspace root
```

## Useful Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server |
| `npm run build` | Build all packages |
| `npm run typecheck` | TypeScript type checking |
| `npm run test` | Run tests |
| `npm run lint` | Lint code |
