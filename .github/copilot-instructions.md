# GitHub Copilot Instructions — WellPlanned

## Project Summary
WellPlanned is a TypeScript/Node.js API that combines a constraint-based scheduling engine with AI (OpenAI GPT-4o) to distribute user tasks across days, accounting for basic human needs. The core scheduler is pure math; AI handles natural-language decomposition and conversation.

---

## Stack
- **Language**: TypeScript 5, strict mode
- **Runtime**: Node.js 20+
- **Framework**: Express 4
- **Validation**: Zod
- **Database**: SQLite via `better-sqlite3` (synchronous)
- **AI**: OpenAI Node SDK, GPT-4o with function calling
- **Testing**: Vitest + Supertest
- **Date math**: date-fns

---

## Coding Conventions

### General
- All files use `.ts` extension, `kebab-case` naming
- Strict TypeScript — no `any`, explicit return types on all exports
- Errors as values in `src/core/` and `src/utils/`; typed throws in `src/ai/`; structured JSON errors in `src/api/`

### Imports
- Use path aliases: `@core/`, `@ai/`, `@models/`, `@utils/`, `@api/`
- Layer dependency rule: `api → core`, `api → ai`, `core → utils` — no reverse or cross-layer imports

### Express Routes
- Thin controllers only: validate with Zod → call service → return JSON
- No business logic in route handlers
- All errors caught by global error middleware in `src/api/middleware/error-handler.ts`

### OpenAI / AI Code
- All calls go through `src/ai/client.ts` — never instantiate OpenAI directly in other files
- Always use function calling (structured output) — never parse freeform text
- Prompt templates are typed functions in `src/ai/prompts/` — never inline prompt strings

### Database
- All queries in `src/models/` — never write SQL outside this directory
- `better-sqlite3` is synchronous — do not use `async/await` with it
- Dates stored as ISO 8601 strings

---

## Testing Conventions

- Framework is **Vitest** (not Jest) — use `vi.mock()`, `vi.fn()`, `vi.spyOn()`
- **Always write the failing test first** before implementing
- Mock OpenAI at `src/ai/client.ts` boundary
- Use in-memory SQLite (`:memory:`) for integration tests
- One `describe` block per unit; one `it` per behaviour; name tests as sentences

---

## What Copilot Should NOT Do

- Do not suggest `any` types — suggest `unknown` with type guards instead
- Do not write business logic in Express route handlers
- Do not add raw SQL outside of `src/models/`
- Do not generate tests that make real network calls
- Do not suggest removing existing tests
- Do not add inline TODO comments — these go in `TODO.md` instead
- Do not suggest `console.log` — use the structured logger at `src/utils/logger.ts`
- Do not refactor code outside the scope of the current task
