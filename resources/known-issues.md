# Known Issues

Active bugs, limitations, and workarounds for AgentTailor.

<!-- Entries added by quality gate failures and /sync-docs -->

## Active Issues

### Code chunking for programming languages
- **Location**: `server/src/services/chunking/chunk-engine.ts:70`
- **Type**: Limitation
- **Description**: Code files fall through to paragraph-based chunking. A `TODO` comment marks the location for future language-aware chunking (AST-based splitting by function/class boundaries).
- **Impact**: Low — code files are still chunked, just not optimally by semantic boundaries.

### Stale feature branches
- **Type**: Cleanup
- **Description**: 18 merged feature branches (feat/T001 through feat/T040) still exist locally. All are merged to main and can be safely deleted.
- **Impact**: None — cosmetic only.

## Resolved

_None yet._
