# WellPlanned — Task Breakdown

## How to Use This File

Strict workflow per task:
1. **Red phase** — write failing tests that define the expected behaviour
2. **Green phase** — implement the minimum code to pass all tests
3. **Review** — read the diff manually; verify intent matches implementation
4. **Commit** — descriptive message, reference the task ID
5. **Update context** — if you discovered something non-obvious, update `CLAUDE.md` or `AGENTS.md`
6. **Gate check** — do NOT start the next Phase until this Phase's tests pass CI and a human has reviewed

---

## Phase 0: Foundation ⬜

> Goal: Working project skeleton with green CI before any feature code.

- [ ] `P0-01` — Initialise `package.json` with Node 20 / TypeScript 5; add `tsconfig.json`
- [ ] `P0-02` — Add Vitest; write and pass a smoke test (`1 + 1 === 2`) — confirms test runner works
- [ ] `P0-03` — Add ESLint + Prettier; enforce no-unused-vars, strict TS, consistent formatting
- [ ] `P0-04` — Add Express skeleton with `/health` endpoint; integration test confirms 200
- [ ] `P0-05` — Add Zod for request validation; unit-test a sample schema
- [ ] `P0-06` — Add SQLite (better-sqlite3) + migration runner; smoke test creates and queries a table
- [ ] `P0-07` — Add `.env.example` (OPENAI_API_KEY, DATABASE_URL, PORT, NODE_ENV)
- [ ] `P0-08` — GitHub Actions CI: install → lint → test on push to `main` and on PRs
- [ ] `P0-09` — Human review of all config files (CLAUDE.md, AGENTS.md, spec.md, TODO.md)

**Evidence gate**: CI green badge on `main` ✅ before Phase 1

---

## Phase 1: Human-Needs Model ⬜

> Goal: A pure math engine that calculates available productive capacity for any day.

- [ ] `P1-01` — Write unit tests for `DayProfile`: given sleep hours, meal blocks, exercise, transitions → returns `availableMinutes`
- [ ] `P1-02` — Implement `DayProfile` class in `src/core/day-profile.ts`
- [ ] `P1-03` — Write unit tests for `WeekCapacity`: given a date range + DayProfile config → returns per-day capacity map
- [ ] `P1-04` — Implement `WeekCapacity` in `src/core/week-capacity.ts`
- [ ] `P1-05` — Write edge-case tests: zero sleep (error), negative blocks (error), capacity = 0 (warn), weekend overrides
- [ ] `P1-06` — Implement validation + error handling; all edge-case tests pass
- [ ] `P1-07` — API endpoint `GET /capacity?from=&to=` returns capacity map as JSON; integration tests
- [ ] `P1-08` — Manual test: call endpoint with a two-week range; verify output makes sense

**Evidence gate**: All unit + integration tests pass; capacity numbers reviewed by human ✅

---

## Phase 2: Task Decomposition Engine ⬜

> Goal: Accept a high-level goal, decompose it into chunks with time estimates.

- [ ] `P2-01` — Write unit tests for `TaskChunk` model (title, estimateMinutes, tags, dependencies)
- [ ] `P2-02` — Implement `TaskChunk` model + Zod schema
- [ ] `P2-03` — Write integration tests for `DecompositionService`: mock OpenAI → verify structured output matches schema
- [ ] `P2-04` — Implement `DecompositionService` in `src/ai/decomposition.ts` using GPT-4o function calling
- [ ] `P2-05` — Prompt engineering: system prompt enforces realistic time estimates (no chunk > 90 min), uses Pomodoro-friendly boundaries
- [ ] `P2-06` — Write tests for fallback: if AI returns invalid schema → `DecompositionService` retries once then throws structured error
- [ ] `P2-07` — API endpoint `POST /goals` accepts `{title, dueDate, context}` → returns `TaskChunk[]`; integration tests
- [ ] `P2-08` — Manual test with 3 real goals; verify chunk quality

**Evidence gate**: All tests pass; AI output reviewed for quality on real inputs ✅

---

## Phase 3: Scheduler — Constraint-Based Distribution ⬜

> Goal: Slot `TaskChunk[]` into available capacity slots, respecting due dates and dependencies.

- [ ] `P3-01` — Write unit tests for `Scheduler.distribute()`: given chunks + capacity map → returns `ScheduledItem[]` with assigned dates
- [ ] `P3-02` — Implement greedy-first `Scheduler` in `src/core/scheduler.ts` (fit chunks into earliest available slot)
- [ ] `P3-03` — Write tests for overload scenario: total chunk minutes > available capacity → returns `OverloadWarning[]`
- [ ] `P3-04` — Implement overload detection; `ScheduleResult` includes `warnings: OverloadWarning[]`
- [ ] `P3-05` — Write tests for dependency ordering: chunk B depends on A → B is never scheduled before A
- [ ] `P3-06` — Implement topological sort for dependencies
- [ ] `P3-07` — Write tests for due-date urgency: chunks closer to due date get priority in scheduling
- [ ] `P3-08` — Implement urgency weighting
- [ ] `P3-09` — API endpoint `POST /schedule` accepts `{chunks, capacityConfig, startDate, endDate}` → returns `ScheduleResult`; integration tests
- [ ] `P3-10` — Manual test: schedule a realistic week; inspect the output timeline

**Evidence gate**: All tests pass including overload + dependency edge cases; output reviewed ✅

---

## Phase 4: AI Calendar Manager — Conversational Interface ⬜

> Goal: Natural-language "what do you want to achieve?" loop that produces a full schedule.

- [ ] `P4-01` — Write integration tests for `PlanningSession`: multi-turn conversation that yields a `ScheduleResult`
- [ ] `P4-02` — Implement `PlanningSession` in `src/ai/planning-session.ts` (stateful conversation, tool calls)
- [ ] `P4-03` — System prompt: friendly coach persona, asks clarifying questions, infers due dates from natural language
- [ ] `P4-04` — Write tests for natural-date parsing: "by end of next week", "before Friday" → ISO date
- [ ] `P4-05` — Implement `parseDueDate()` using date-fns + AI fallback
- [ ] `P4-06` — `POST /plan/start` → begins session; `POST /plan/:id/message` → continues; `GET /plan/:id/schedule` → final result
- [ ] `P4-07` — Session persistence in SQLite; tests confirm resume after restart
- [ ] `P4-08` — Manual test: full planning conversation for a real upcoming week

**Evidence gate**: Full conversation flow tested end-to-end; reviewed by human ✅

---

## Phase 5: Daily Brief & Future View ⬜

> Goal: Surface the schedule in a useful daily format.

- [ ] `P5-01` — `GET /brief/today` endpoint → returns today's scheduled tasks, available buffer, and overload warnings
- [ ] `P5-02` — `GET /schedule/week?from=` → returns full week view with per-day breakdowns
- [ ] `P5-03` — Write tests for "what's at risk" logic: tasks within 24h of due date with incomplete deps flagged
- [ ] `P5-04` — Implement at-risk flagging in `src/core/risk-detector.ts`
- [ ] `P5-05` — CLI command `npm run brief` → prints today's plan to stdout (good for terminal lovers)
- [ ] `P5-06` — Manual test: run brief and review output quality

**Evidence gate**: Endpoints tested; brief output manually reviewed ✅

---

## Phase 6: Polish & Harden ⬜

- [ ] `P6-01` — Add rate-limiting middleware (express-rate-limit) to all AI endpoints
- [ ] `P6-02` — Add request logging (pino) with structured JSON output
- [ ] `P6-03` — Add cost tracking for OpenAI calls; log tokens used per request
- [ ] `P6-04` — Write load test for scheduler: 500 tasks, 30-day window, must complete < 200ms
- [ ] `P6-05` — Add `GET /health/deep` endpoint: checks DB + OpenAI connectivity
- [ ] `P6-06` — Security audit: validate all user inputs, no raw SQL, no exposed stack traces
- [ ] `P6-07` — Update all documentation; ensure CLAUDE.md / AGENTS.md reflect final commands

---

## Phase 7: Ship ⬜

- [ ] `P7-01` — Dockerfile; `docker build` + `docker run` smoke test
- [ ] `P7-02` — Deploy to Railway / Fly.io / Render; production smoke test
- [ ] `P7-03` — Add uptime monitoring (Better Uptime or similar)
- [ ] `P7-04` — Write `CHANGELOG.md` v0.1.0 entry

---

## Parking Lot 🅿️

> Ideas that are not in scope yet but worth keeping:

- Mobile PWA frontend with drag-to-reorder
- Calendar sync (Google Calendar, iCal)
- Recurring tasks and habits
- Team mode: shared capacity planning
- Energy-level modelling (morning = deep work, afternoon = admin)
- Pomodoro timer integration
- Weekly retrospective: planned vs actual

---

## Lessons Learned 📝

> Update this section whenever you discover something non-obvious. This is the project's institutional memory.

- _(empty — first session)_
