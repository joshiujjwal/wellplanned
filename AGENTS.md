# AGENTS.md — WellPlanned

Instructions for AI coding agents (OpenAI Codex, GitHub Copilot, etc.).

---

## Setup

```bash
# 1. Install Node 20+ (check: node -v)
# 2. Install dependencies
npm install

# 3. Copy and fill in env vars
cp .env.example .env
# Required: OPENAI_API_KEY, DATABASE_URL=./dev.db, PORT=3000, NODE_ENV=development

# 4. Verify tests pass before touching anything
npm test
```

---

## Code Style

### TypeScript
- Strict mode: `noImplicitAny`, `strictNullChecks`, `exactOptionalPropertyTypes` all ON
- Explicit return types on all exported functions
- No `any` — use `unknown` and narrow with type guards
- Prefer `type` over `interface` unless you need `extends`

### File & Naming Conventions
- Files: `kebab-case.ts`
- Classes / types: `PascalCase`
- Functions / variables: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE`
- Test files: `<subject>.test.ts` co-located with source OR under `tests/`

### Error Handling
- **Core layer** (`src/core/`, `src/utils/`): return `Result<T, E>` — never throw
- **AI layer** (`src/ai/`): throw typed errors (`AiDecompositionError`, etc.)
- **API layer** (`src/api/`): catch all errors in middleware; return structured JSON `{ error: { code, message } }`
- Never expose stack traces in API responses

### Imports
- Absolute imports from `src/` using TypeScript path aliases (`@core/`, `@ai/`, `@models/`, `@utils/`)
- No circular imports between layers: `api → core ← (no ai dependency)`, `api → ai`, `core → utils`

---

## Testing

### Framework
- **Vitest** (not Jest) — APIs differ; check Vitest docs
- `npm test` — runs all tests
- `npm run test:unit` — unit only (fast, no I/O)
- `npm run test:integration` — needs in-memory SQLite, mocked OpenAI

### Red/Green TDD — REQUIRED
1. Write the failing test FIRST — confirm it fails for the right reason
2. Write the minimum code to make it pass
3. Refactor only after it is green

### Mocking
- Mock OpenAI at the `src/ai/client.ts` boundary using `vi.mock()`
- Use in-memory SQLite for integration tests (`:memory:` database URL)
- Never make real network calls in tests

### Coverage
- All exported functions in `src/core/` and `src/utils/` must have unit tests
- Integration tests must cover every API endpoint's happy path + main error cases
- Do NOT delete tests. If behaviour changes, update the test.

### Writing Good Tests
```typescript
// ✅ Good — specific, explains intent, one assertion per test
it('returns 0 availableMinutes when all blocks exceed focus window', () => {
  const profile = new DayProfile({ focusStart: '09:00', focusEnd: '10:00', mealBlocks: 3, exerciseMin: 30 });
  expect(profile.availableMinutes).toBe(0);
});

// ❌ Bad — vague name, tests everything at once
it('works correctly', () => { ... });
```

---

## PR Instructions

Before opening a PR:
1. `npm test` — all tests pass
2. `npm run lint` — no lint errors
3. `npm run typecheck` — no type errors
4. Manually test the feature with at least one real input

PR description MUST include:
- **What**: one-sentence summary
- **Why**: link to the TODO item (e.g., `P2-04`)
- **Evidence**: paste test output (`npm test` summary) + screenshot or curl output of manual test
- **Checklist**: confirm you did not remove any tests, did not add `any` types, did not hardcode secrets

Keep PRs small and focused. One TODO task = one PR where possible.

---

## What NOT to Do

- Do NOT refactor code that is not related to your current task
- Do NOT upgrade dependencies without a dedicated task for it
- Do NOT skip the red phase — write the failing test first, always
- Do NOT add `console.log` debug statements to committed code (use the logger in `src/utils/logger.ts`)
- Do NOT store secrets in code or commit `.env` files
- Do NOT add `TODO:` comments to the code — instead, add the task to `TODO.md`
