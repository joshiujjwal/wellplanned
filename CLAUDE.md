# CLAUDE.md — WellPlanned

> Context file for AI coding assistants. Keep under 200 lines. Update when you discover something non-obvious.

---

## What This Project Does

WellPlanned is an AI calendar manager that asks "what do you want to achieve?" and produces a **math-backed schedule** — distributing tasks over days while reserving time for basic human needs (sleep, meals, exercise, transitions). It uses OpenAI GPT-4o to decompose goals into atomic chunks, then slots those chunks into available capacity using a constraint-based scheduler.

---

## Commands

```bash
# Install dependencies
npm install

# Dev server (hot reload via tsx watch)
npm run dev

# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run only unit tests
npm run test:unit

# Run only integration tests
npm run test:integration

# Lint (ESLint)
npm run lint

# Format (Prettier)
npm run format

# Type-check without emitting
npm run typecheck

# Build for production
npm run build

# Start production server
npm start

# Daily brief in terminal
npm run brief
```

> **IMPORTANT**: Always run `npm test` BEFORE making any changes. Confirm you are starting from green.

---

## Directory Map

```
src/
  api/          Express routes + controllers. One file per resource group.
  core/         Pure scheduling logic — NO Express, NO AI, NO DB dependencies.
                This is the heart of the app; keep it pure and fast.
  ai/           OpenAI integration. Prompt templates, function schemas, retry logic.
  models/       SQLite schema + typed query helpers (better-sqlite3).
  utils/        Time math, date parsing, capacity calculations. Pure functions only.

tests/
  unit/         Tests for src/core/ and src/utils/ — no DB, no network, no AI.
  integration/  Tests for src/api/ — real SQLite in-memory, mocked OpenAI.
  e2e/          Full flow tests — real server, real DB, mocked OpenAI.

docs/
  spec.md       The source of truth for features. Read before implementing anything.
  adr/          Architecture Decision Records. Add one when making a significant choice.
```

---

## Architecture Notes

### Core Scheduling Engine (`src/core/`)
- **Pure functions only** — no I/O, no side effects, no dependencies outside `src/utils/`
- `DayProfile` → models a single day's available capacity
- `WeekCapacity` → maps a date range to per-day available minutes
- `Scheduler` → slots `TaskChunk[]` into capacity using greedy-first + topological sort
- `RiskDetector` → identifies tasks at risk based on proximity to due date

### AI Layer (`src/ai/`)
- All OpenAI calls go through `src/ai/client.ts` (single entry point, easy to swap)
- GPT-4o **function calling** (structured output) for decomposition — never parse freeform AI text
- Every prompt template lives in `src/ai/prompts/` as a typed function, not a loose string
- Retry once on schema validation failure, then throw `AiDecompositionError`

### API Layer (`src/api/`)
- Zod schemas validate ALL incoming request bodies — fail fast with structured errors
- Route handlers are thin: validate → call service → return result
- No business logic in route handlers

### Database (`src/models/`)
- `better-sqlite3` (synchronous) — simplifies code, fine for this workload
- Migrations in `src/models/migrations/` numbered sequentially (`001_init.sql`, etc.)
- Never write raw SQL outside of `src/models/` files

---

## Key Conventions

- **TypeScript strict mode** — `noImplicitAny`, `strictNullChecks`, `exactOptionalPropertyTypes`
- **Errors as values** in the core layer: return `{ ok: false, error: ... }` not throw
- **Throw at the edges**: API layer and AI layer may throw; core never does
- Dates stored as ISO 8601 strings in DB; manipulated with `date-fns` in code
- All times in **UTC** internally; format for display at the presentation layer
- File naming: `kebab-case.ts` for files, `PascalCase` for classes/types, `camelCase` for functions

---

## Environment Variables

See `.env.example` for all variables. Required for dev:

```
OPENAI_API_KEY=sk-...
DATABASE_URL=./dev.db
PORT=3000
NODE_ENV=development
```

---

## Workflow

1. Read `TODO.md` — find the next unchecked task in the current Phase
2. Run `npm test` — confirm you start from green
3. Write the failing test(s) for that task (red phase)
4. Implement until tests pass (green phase)
5. Run `npm run lint && npm run typecheck` — fix any issues
6. Commit with message: `[P<phase>-<task>] <what you did>`
7. If you learned something non-obvious, update the **Lessons Learned** section in `TODO.md` and this file

---

## Gotchas

- `better-sqlite3` is synchronous — do NOT wrap in `async/await` or you will get confusing errors
- The scheduling engine assumes all times are in the same timezone — mixing timezones will silently produce wrong schedules
- OpenAI function calling returns `arguments` as a **JSON string**, not a parsed object — always `JSON.parse()`
- Vitest and Jest have different `mock` APIs — we use Vitest; check docs before using `jest.fn()`
