# Development Agents

## Agent Types

| Agent Type | Model | Tasks | Description |
|-----------|-------|-------|-------------|
| **scaffold-agent** | sonnet | T001 | Monorepo setup, Docker, build configs |
| **backend-agent** | sonnet | T002, T003, T004, T007, T008, T038 | Express server, DB, APIs, middleware |
| **pipeline-agent** | sonnet | T005, T006 | Document processing, chunking, embeddings, vectors |
| **intelligence-agent** | sonnet | T009–T015 | Context intelligence engine (THE CORE) |
| **search-agent** | sonnet | T016–T018 | Web search integration, citations |
| **frontend-agent** | sonnet | T019–T024, T037, T039 | React dashboard, pages, components |
| **extension-agent** | sonnet | T025–T030 | Chrome extension, content scripts, injection |
| **integration-agent** | sonnet | T031–T035 | MCP server, Custom GPT, platform adapters |
| **polish-agent** | sonnet | T036 | Quality scoring, polish |
| **test-agent** | sonnet | T040 | E2E tests, CI/CD, production deployment |

## Batch Execution Plan

### Batch 1 (1 agent)
| Task | Agent |
|------|-------|
| T001 | scaffold-agent |

### Batch 2 (max 4 parallel)
| Task | Agent |
|------|-------|
| T002 | backend-agent |
| T005 | pipeline-agent |
| T009 | intelligence-agent |
| T012 | intelligence-agent |

_Also available: T014, T016, T025, T039 (start if capacity allows)_

### Batch 3 (max 4 parallel)
| Task | Agent |
|------|-------|
| T003 | backend-agent |
| T006 | pipeline-agent |
| T014 | intelligence-agent |
| T016 | search-agent |

_Also available: T025, T026, T027, T028, T030, T039_

### Batch 4 (max 4 parallel)
| Task | Agent |
|------|-------|
| T004 | backend-agent |
| T008 | backend-agent |
| T019 | frontend-agent |
| T038 | backend-agent |

_Also available: T025, T039_

### Batch 5 (max 4 parallel)
| Task | Agent |
|------|-------|
| T007 | backend-agent |
| T020 | frontend-agent |
| T010 | intelligence-agent |
| T025 | extension-agent |

### Batch 6 (max 5 parallel)
| Task | Agent |
|------|-------|
| T021 | frontend-agent |
| T022 | frontend-agent |
| T024 | frontend-agent |
| T011 | intelligence-agent |
| T013 | intelligence-agent |

_Also available: T026, T027, T028, T030_

### Batch 7 (max 4 parallel)
| Task | Agent |
|------|-------|
| T015 | intelligence-agent |
| T017 | search-agent |
| T026 | extension-agent |
| T027 | extension-agent |

_Also available: T028, T030_

### Batch 8 (max 5 parallel)
| Task | Agent |
|------|-------|
| T023 | frontend-agent |
| T018 | search-agent |
| T028 | extension-agent |
| T030 | extension-agent |
| T031 | integration-agent |

_Also available: T034, T036, T039_

### Batch 9 (max 4 parallel)
| Task | Agent |
|------|-------|
| T029 | extension-agent |
| T032 | integration-agent |
| T033 | integration-agent |
| T034 | integration-agent |

_Also available: T035, T036, T037, T039_

### Batch 10 (max 4 parallel)
| Task | Agent |
|------|-------|
| T035 | integration-agent |
| T036 | polish-agent |
| T037 | frontend-agent |
| T039 | frontend-agent |

### Batch 11 (1 agent)
| Task | Agent |
|------|-------|
| T040 | test-agent |

## Coverage Matrix

| Agent Type | Tasks | Phase Coverage |
|-----------|-------|----------------|
| scaffold-agent | 1 | Phase 1 |
| backend-agent | 6 | Phase 1, 8 |
| pipeline-agent | 2 | Phase 1 |
| intelligence-agent | 7 | Phase 1, 2 |
| search-agent | 3 | Phase 3 |
| frontend-agent | 8 | Phase 4, 8 |
| extension-agent | 6 | Phase 5 |
| integration-agent | 5 | Phase 6, 7 |
| polish-agent | 1 | Phase 8 |
| test-agent | 1 | Phase 8 |
