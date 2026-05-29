# WellPlanned — Feature Specification

**Status**: Draft v0.1  
**Last updated**: 2025-01  
**Owner**: TBD

---

## 1. Overview

### Problem Statement

Most task managers ask you to assign tasks to specific times yourself — a cognitively expensive process that ignores how time actually works. People forget to reserve time for sleep, meals, commutes, and recovery. Deadlines slip not because people are lazy, but because their schedule had impossible maths from the start.

**WellPlanned** inverts this: instead of asking "when will you do this?", it asks "what do you want to achieve?" — then uses constraint-based scheduling to distribute work across realistic, human-needs-aware capacity.

### Target Users

- Knowledge workers managing multiple concurrent projects
- Anyone who has ever made a schedule that fell apart by Tuesday
- People who want a brutally honest answer to "can I actually do all this?"

---

## 2. Functional Requirements

### 2.1 Human-Needs Model

- [ ] System maintains a configurable `DayProfile` per user
  - `sleepHours` (default: 8)
  - `mealBlocks` (default: 3 × 30 min)
  - `exerciseMinutes` (default: 30)
  - `transitionMinutes` (default: 30, covers commute / wind-down)
  - `focusStartHour` / `focusEndHour` (default: 09:00 – 18:00)
- [ ] `availableMinutes = (focusEndHour - focusStartHour) × 60 - mealBlocks - exerciseMinutes - transitionMinutes`
- [ ] Weekend days have a separate profile (default: 50% reduced capacity)
- [ ] Capacity never goes negative; minimum is 0

### 2.2 Goal & Task Intake

- [ ] User can submit a goal in natural language: title, optional due date, optional context/notes
- [ ] System uses AI (GPT-4o) to decompose goal into `TaskChunk[]`
  - Each chunk: `title`, `estimateMinutes`, `tags[]`, `dependsOn[]` (chunk IDs)
  - No single chunk exceeds 90 minutes (enforced by prompt + validation)
  - Chunks respect declared dependencies (sequential ordering)
- [ ] System infers due date from natural-language expressions ("by end of next week")
- [ ] User can edit, merge, split, or delete chunks before scheduling

### 2.3 Constraint-Based Scheduler

- [ ] Given `TaskChunk[]` + `capacityMap` + `startDate` + `endDate`:
  - Returns `ScheduledItem[]`: each chunk assigned to a specific day + time slot
  - Respects dependency ordering (topological sort)
  - Prioritises by urgency (due date proximity)
  - Fills days greedily from earliest available slot
- [ ] Returns `OverloadWarning[]` when total chunk time exceeds available capacity
  - Warning includes: which days are over-capacity, by how many minutes
  - Suggests extending deadline or reducing scope
- [ ] Scheduler runs synchronously and must complete in < 200ms for up to 500 chunks over 30 days

### 2.4 Conversational Planning Interface

- [ ] `POST /plan/start` — create a new planning session
- [ ] `POST /plan/:id/message` — send a message; AI responds with clarifying questions or confirms the plan
- [ ] `GET /plan/:id/schedule` — retrieve the current draft schedule
- [ ] AI asks clarifying questions when:
  - Due date is missing or ambiguous
  - Goal is too vague to decompose (e.g., "be more productive")
  - Estimated scope clearly exceeds available time
- [ ] Session state persists in SQLite; sessions survive server restarts

### 2.5 Daily Brief

- [ ] `GET /brief/today` — returns:
  - Today's scheduled chunks (ordered by time slot)
  - Available buffer minutes remaining today
  - `atRisk[]`: tasks within 24h of due date with incomplete predecessor chunks
  - `overloadWarnings[]` if applicable
- [ ] `GET /schedule/week?from=YYYY-MM-DD` — full 7-day view
- [ ] CLI: `npm run brief` prints today's plan to stdout

---

## 3. Non-Functional Requirements

- [ ] **Latency**: Scheduler (non-AI path) < 200ms for p95
- [ ] **AI cost**: Track token usage per request; warn if single decomposition exceeds 2000 tokens
- [ ] **Reliability**: `/health` responds 200 even if OpenAI is down (graceful degradation)
- [ ] **Security**: All user input validated with Zod; no raw SQL; stack traces not exposed in API responses
- [ ] **Portability**: Runs with `npm run dev` on a fresh machine with only Node 20 + an `.env` file

---

## 4. Data Model

### `UserProfile`
```
id            TEXT  PRIMARY KEY
sleepHours    REAL  DEFAULT 8
mealBlocks    INT   DEFAULT 3       -- count of 30-min meal blocks
exerciseMin   INT   DEFAULT 30
transitionMin INT   DEFAULT 30
focusStart    TEXT  DEFAULT '09:00'
focusEnd      TEXT  DEFAULT '18:00'
createdAt     TEXT
updatedAt     TEXT
```

### `Goal`
```
id          TEXT  PRIMARY KEY
userId      TEXT  REFERENCES UserProfile
title       TEXT  NOT NULL
dueDate     TEXT  -- ISO 8601 date
context     TEXT  -- free-form notes
status      TEXT  -- 'draft' | 'scheduled' | 'done'
createdAt   TEXT
```

### `TaskChunk`
```
id              TEXT  PRIMARY KEY
goalId          TEXT  REFERENCES Goal
title           TEXT  NOT NULL
estimateMin     INT   NOT NULL
tags            TEXT  -- JSON array
dependsOn       TEXT  -- JSON array of TaskChunk.id
status          TEXT  -- 'pending' | 'in-progress' | 'done'
scheduledDate   TEXT  -- ISO 8601 date (null if unscheduled)
scheduledSlot   TEXT  -- 'HH:MM' start time (null if unscheduled)
```

### `PlanningSession`
```
id          TEXT  PRIMARY KEY
userId      TEXT  REFERENCES UserProfile
status      TEXT  -- 'active' | 'completed' | 'abandoned'
messages    TEXT  -- JSON array of {role, content}
scheduleId  TEXT  -- FK to resulting Schedule
createdAt   TEXT
updatedAt   TEXT
```

---

## 5. API Design

| Method | Path | Description |
|---|---|---|
| `GET` | `/health` | Liveness check |
| `GET` | `/health/deep` | DB + OpenAI connectivity check |
| `GET` | `/capacity` | `?from=&to=` — returns per-day available minutes |
| `POST` | `/goals` | Decompose a goal into TaskChunks |
| `GET` | `/goals/:id` | Get goal with its chunks |
| `POST` | `/schedule` | Distribute chunks over a date range |
| `POST` | `/plan/start` | Start a conversational planning session |
| `POST` | `/plan/:id/message` | Send a message in a session |
| `GET` | `/plan/:id/schedule` | Get the draft schedule for a session |
| `GET` | `/brief/today` | Today's daily brief |
| `GET` | `/schedule/week` | `?from=` — 7-day schedule view |

---

## 6. Test Plan

### Unit Tests (`tests/unit/`)
- `DayProfile`: all input combinations, edge cases (zero sleep, max capacity)
- `WeekCapacity`: multi-week range, weekend overrides
- `Scheduler.distribute()`: greedy fill, dependency order, overload detection
- `parseDueDate()`: natural language → ISO date (10+ fixtures)
- Zod schemas: valid + invalid inputs for every request type

### Integration Tests (`tests/integration/`)
- `POST /goals`: mock OpenAI → verify response matches schema
- `POST /schedule`: real scheduler, no AI dependency
- `POST /plan/start` + `POST /plan/:id/message`: mock AI conversation flow
- `GET /brief/today`: DB-backed test with seeded data

### Edge Cases
- Goal with no feasible schedule (all days full) → structured `OverloadWarning`
- Circular dependency in chunks → validation error, not infinite loop
- AI returns malformed JSON → retry once, then structured error
- Zero available days in date range → error, not crash
- Chunk estimate of 0 minutes → validation error

---

## 7. Open Questions

1. **Auth**: No auth in v0.1 (single-user, local). What auth strategy for multi-user? (JWT? OAuth?)
2. **Frontend**: Build a React frontend or focus on API + CLI first?
3. **AI model choice**: GPT-4o for quality vs GPT-4o-mini for cost — needs real-world benchmarking
4. **Scheduling algorithm**: Greedy-first is simple but not optimal. Worth exploring constraint programming (e.g., `javascript-cp`) later?
5. **Time zones**: All times stored in UTC internally? User profile stores timezone?
6. **Recurring tasks**: Habits / dailies are a natural extension — design the data model to allow this without a breaking migration
