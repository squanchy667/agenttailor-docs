# T014 — Context Window Manager

| Field | Value |
|-------|-------|
| Phase | 2 — Context Intelligence Engine |
| Agent Type | intelligence-agent |
| Depends On | T001 |
| Blocks | T015 |
| Branch | `feat/T014-context-window-manager` |
| Status | PENDING |

## Objective
Track token usage across assembled context and enforce model-specific limits (128K GPT-4, 200K Claude). Allocate token budget across context sections (system prompt, project docs, web results, examples) to maximize information density within the target model's context window.

## Context
Different AI models have different context window sizes, and not all of that window should be consumed by injected context — the user needs room for their conversation. The Context Window Manager is the accountant of the tailoring pipeline: it knows exactly how many tokens are used, enforces hard limits, and intelligently allocates budget across sections. Without this, we risk either under-utilizing the context window (wasting potential) or exceeding it (causing errors or truncation).

## Subtasks
1. Create `server/src/lib/tokenCounter.ts`:
   - Install and configure `tiktoken` (or `js-tiktoken`) for accurate token counting.
   - `countTokens(text, model?)` — Count tokens for a given text. Default to cl100k_base encoding (GPT-4/Claude compatible).
   - `countTokensBatch(texts[])` — Count tokens for multiple texts efficiently.
   - `estimateTokens(text)` — Fast approximate count (chars / 4) for when exact count is unnecessary.
   - Cache token counts to avoid recomputation.
2. Create `shared/src/schemas/tokenBudget.ts` — Zod schemas and constants:
   - `ModelConfig` — modelId (string), maxContextTokens (number), reservedForResponse (number), reservedForConversation (number).
   - `MODEL_CONFIGS` — Predefined configs:
     - GPT-4: 128K context, 4K response, 8K conversation reserve.
     - GPT-4o: 128K context, 16K response, 8K conversation reserve.
     - Claude Sonnet: 200K context, 8K response, 16K conversation reserve.
     - Claude Opus: 200K context, 8K response, 16K conversation reserve.
   - `TokenBudget` — totalAvailable, allocations (map of section name to token count), used (map of section name to actual usage), remaining.
   - `BudgetAllocationStrategy` — proportional (percentage-based) or priority (fill highest priority first).
3. Create `server/src/services/intelligence/contextWindowManager.ts`:
   - `createBudget(targetPlatform, model?)`:
     a. Look up ModelConfig for the target.
     b. Calculate available tokens: maxContext - reservedForResponse - reservedForConversation.
     c. Allocate across sections with default proportions: system prompt (5%), project docs (50%), web results (20%), examples (15%), metadata/formatting (10%).
     d. Return TokenBudget object.
   - `allocateBudget(totalTokens, sections, strategy)` — Custom allocation with either proportional or priority strategy.
   - `trackUsage(budget, section, tokenCount)` — Update usage tracking. Return remaining budget for the section.
   - `isWithinBudget(budget)` — Check if total usage is within limits.
   - `rebalance(budget)` — If some sections are under-used, redistribute their unused tokens to over-budget sections by priority.
   - `getBudgetReport(budget)` — Human-readable summary of allocation vs. usage per section.
4. Write unit tests covering:
   - Token counting accuracy against known tiktoken outputs.
   - Budget creation for each model config.
   - Allocation strategies (proportional vs. priority).
   - Rebalancing logic.
   - Edge cases: very small budgets, single-section usage.

## Files to Create/Modify
- `server/src/services/intelligence/contextWindowManager.ts` — Budget creation, allocation, tracking, and rebalancing
- `server/src/lib/tokenCounter.ts` — Accurate token counting with tiktoken
- `shared/src/schemas/tokenBudget.ts` — Model configs, token budget schemas, allocation strategies

## Acceptance Criteria
- [ ] Token counter produces accurate counts matching tiktoken for GPT-4 encoding
- [ ] Model configs correctly define limits for GPT-4, GPT-4o, Claude Sonnet, and Claude Opus
- [ ] Budget creation reserves tokens for response and conversation
- [ ] Default allocation gives 50% to project docs, 20% to web results, 15% to examples
- [ ] Proportional allocation distributes tokens by percentage across sections
- [ ] Priority allocation fills highest-priority sections first
- [ ] `trackUsage` correctly updates used counts and remaining budget
- [ ] `isWithinBudget` returns false when any section exceeds its allocation
- [ ] `rebalance` redistributes unused tokens from under-used sections to over-budget ones
- [ ] Budget report clearly shows allocation vs. usage per section
- [ ] Fast `estimateTokens` is within 10% of accurate count for typical text
- [ ] Token count caching prevents recomputation of the same text
