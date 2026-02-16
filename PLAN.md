# AgentTailor — Master Plan

## Vision

AgentTailor is a cross-platform AI agent tailoring add-on that enables users to design, customize, and deploy specialized AI agent configurations for both ChatGPT and Claude. It provides a unified visual interface for composing agent behaviors, tool integrations, and conversation patterns — then exporting platform-specific configurations. The tool democratizes AI agent creation by making it accessible, repeatable, and portable.

## Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Frontend | React + TypeScript + Vite | SPA with fast HMR |
| Styling | Tailwind CSS | Utility-first responsive design |
| Backend | Express.js + TypeScript | API server and configuration processing |
| Validation | Zod | Schema-first validation with inferred types |
| Testing | Vitest | Unit and integration tests |

## Key Features

- **Visual Agent Builder** — Drag-and-drop interface for composing agent system prompts, personality traits, and behavioral rules
- **Tool Configuration** — Define and wire up tools/functions with schema validation for each platform's format
- **Cross-Platform Export** — Generate platform-specific configs (ChatGPT Custom GPTs, Assistants API, Claude Projects, Claude API)
- **Template Library** — Pre-built agent templates for common use cases (customer support, coding assistant, research helper, etc.)
- **Version Control** — Track agent configuration changes over time with diff view
- **Live Preview** — Test agent behavior with simulated conversations before deploying
- **Import/Reverse Engineer** — Import existing agent configs from ChatGPT or Claude to edit and cross-deploy

## Phase Overview

<!-- Phases will be populated by /plan-project -->

_Run `/plan-project AgentTailor` to generate detailed phase breakdown with tasks._

## Architecture

See [System Overview](./architecture/system-overview.md) for detailed architecture documentation.

## Task Board

See [TASK_BOARD.md](./TASK_BOARD.md) for the complete task breakdown with dependency tracking.
