# T009 — Task Analyzer Module

| Field | Value |
|-------|-------|
| Phase | 2 — Context Intelligence Engine |
| Agent Type | intelligence-agent |
| Depends On | T001 |
| Blocks | T015 |
| Branch | `feat/T009-task-analyzer` |
| Status | PENDING |

## Objective
Build the module that parses user input to identify required knowledge domains, task type (coding, writing, analysis, research), complexity level, and key entities. Uses NLP heuristics, keyword matching, and optional LLM classification for ambiguous inputs.

## Context
The Task Analyzer is the first step in the tailoring pipeline. Before we can retrieve relevant context, we need to understand what the user is trying to do. A request like "Implement rate limiting for the API endpoints" should be classified as a coding task in the backend/security domain at medium complexity. This classification drives which documents to search, how many chunks to retrieve, and how to prioritize results.

## Subtasks
1. Create `shared/src/schemas/taskAnalysis.ts` — Types and Zod schemas:
   - `TaskType` enum: CODING, WRITING, ANALYSIS, RESEARCH, DEBUGGING, DESIGN, PLANNING, OTHER.
   - `ComplexityLevel` enum: LOW, MEDIUM, HIGH, EXPERT.
   - `KnowledgeDomain` — string union or extensible enum: FRONTEND, BACKEND, DATABASE, DEVOPS, SECURITY, TESTING, DESIGN, ARCHITECTURE, DOCUMENTATION, BUSINESS, DATA_SCIENCE, GENERAL.
   - `TaskAnalysis` — taskType, complexity, domains (array), keyEntities (extracted nouns/terms), suggestedSearchQueries (array of strings for vector search), estimatedTokenBudget (based on complexity), confidence (0-1 score for classification certainty).
2. Create `server/src/services/intelligence/domainClassifier.ts`:
   - Keyword dictionaries for each domain (e.g., BACKEND: ["api", "endpoint", "server", "express", "middleware", "route"]).
   - `classifyDomains(text)` — Score each domain based on keyword presence and density. Return top N domains above threshold.
   - `detectTaskType(text)` — Classify based on action verbs and patterns ("implement", "build", "fix", "debug", "write", "analyze", "research", "design").
   - `assessComplexity(text, domains)` — Heuristic scoring based on: number of domains involved, presence of integration keywords ("connect", "between", "across"), specificity of requirements.
3. Create `server/src/services/intelligence/taskAnalyzer.ts`:
   - `analyzeTask(input: string): Promise<TaskAnalysis>`:
     a. Run domain classification.
     b. Detect task type.
     c. Assess complexity.
     d. Extract key entities (nouns, proper nouns, technical terms) using basic NLP patterns.
     e. Generate suggested search queries — rephrase the input into 2-4 specific queries optimized for semantic search.
     f. Calculate estimated token budget based on complexity (LOW: 2K, MEDIUM: 4K, HIGH: 8K, EXPERT: 16K).
     g. Calculate confidence score based on keyword match strength and classification clarity.
   - If confidence is below threshold (0.4), optionally invoke LLM for classification refinement (behind feature flag).
4. Write unit tests covering:
   - Domain classification for various input types.
   - Task type detection accuracy.
   - Complexity assessment calibration.
   - Search query generation quality.

## Files to Create/Modify
- `server/src/services/intelligence/taskAnalyzer.ts` — Main task analysis orchestrator
- `server/src/services/intelligence/domainClassifier.ts` — Domain, type, and complexity classification
- `shared/src/schemas/taskAnalysis.ts` — Zod schemas for TaskType, ComplexityLevel, KnowledgeDomain, TaskAnalysis

## Acceptance Criteria
- [ ] "Implement rate limiting for API endpoints" classified as CODING, BACKEND+SECURITY domain, MEDIUM complexity
- [ ] "Write user documentation for the onboarding flow" classified as WRITING, DOCUMENTATION domain, LOW complexity
- [ ] "Debug why WebSocket connections drop after 30 seconds" classified as DEBUGGING, BACKEND domain, MEDIUM complexity
- [ ] "Design a microservices architecture for the payment system" classified as DESIGN, ARCHITECTURE+BACKEND domain, HIGH complexity
- [ ] Key entities are extracted (e.g., "rate limiting", "API endpoints" from the first example)
- [ ] 2-4 suggested search queries are generated per task input
- [ ] Token budget scales with complexity level
- [ ] Confidence score reflects classification certainty (high for clear tasks, low for ambiguous)
- [ ] Multi-domain tasks correctly identify all relevant domains
- [ ] Unit tests pass with at least 80% classification accuracy on test cases
