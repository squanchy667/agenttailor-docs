# AgentTailor

**Cross-platform AI agent tailoring add-on for ChatGPT and Claude**

## Vision

AgentTailor is a browser-based tool that enables users to design, customize, and deploy specialized AI agent configurations across multiple platforms. Instead of manually crafting system prompts and tool definitions for each platform separately, AgentTailor provides a unified interface where users can visually compose agent behaviors, tool integrations, and conversation patterns — then export tailored configurations for both ChatGPT (Custom GPTs / Assistants API) and Claude (Projects / API). The goal is to make AI agent creation accessible, repeatable, and portable across the leading LLM platforms.

## Architecture at a Glance

See [System Overview](./architecture/system-overview.md) for the full architecture.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React, TypeScript, Vite, Tailwind CSS |
| Backend | Express.js, TypeScript |
| Validation | Zod |
| Testing | Vitest |

## Repos

| Repo | Purpose |
|------|---------|
| `agenttailor-docs` | Architecture, specs, task board, test plans |
| `agenttailor` | Source code |

## Documentation

See [SUMMARY.md](./SUMMARY.md) for the full table of contents.

## Task Board

See [TASK_BOARD.md](./TASK_BOARD.md) for the complete task breakdown with agent assignments and parallel execution plan.
