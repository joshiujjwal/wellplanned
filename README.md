# WellPlanned 🗓️

> Plan your day differently and see into the future.

[![Status](https://img.shields.io/badge/status-🚧%20Early%20Development-orange)]()
[![License](https://img.shields.io/badge/license-MIT-blue)]()

WellPlanned is an AI-powered calendar manager that asks "what do you want to achieve?" and builds a **realistic, math-backed schedule** — distributing tasks across days while honouring all basic human needs (sleep, meals, exercise, recovery). Work items are decomposed into small, manageable chunks and slotted into a future timeline you can actually keep.

---

## ✨ Features (Planned)

| Feature | Description |
|---|---|
| Goal intake | Natural-language input: "I want to finish this project by Friday" |
| Task decomposition | AI breaks goals into atomic, time-estimable chunks |
| Human-needs modelling | Reserves blocks for sleep, meals, exercise, transitions |
| Math-based distribution | Remaining capacity spread optimally over available days |
| Future-view calendar | Visual timeline showing scheduled vs available capacity |
| Overload detection | Warns when schedule is unrealistic before it is locked |
| Daily brief | Morning summary of today's plan and what's at risk |

---

## 🛠 Tech Stack

- **Runtime**: Node.js 20+ / TypeScript 5
- **API Layer**: Express + Zod validation
- **AI**: OpenAI GPT-4o (function calling for structured decomposition)
- **Scheduling Engine**: Custom constraint-based solver (`src/core/`)
- **Storage**: SQLite (local dev) / PostgreSQL (production)
- **Frontend**: React 18 + Vite (TBD — API-first for now)
- **Testing**: Vitest + Supertest
- **CI**: GitHub Actions

---

## 🚀 Getting Started

```bash
# Clone
git clone https://github.com/<your-org>/wellplanned.git
cd wellplanned

# Install
npm install

# Configure
cp .env.example .env
# → add OPENAI_API_KEY and DATABASE_URL

# Dev server
npm run dev

# Run tests
npm test

# Lint
npm run lint
```

---

## 📁 Project Structure

```
wellplanned/
├── src/
│   ├── api/          # Express routes & controllers
│   ├── core/         # Scheduling engine, constraint solver
│   ├── ai/           # OpenAI integration, prompt templates
│   ├── models/       # Data models & DB schema
│   └── utils/        # Time helpers, capacity math
├── tests/
│   ├── unit/         # Pure logic tests (core/, utils/)
│   ├── integration/  # API + DB tests
│   └── e2e/          # Full flow tests
├── docs/
│   ├── spec.md       # Feature specification
│   └── adr/          # Architecture Decision Records
└── .github/
    ├── copilot-instructions.md
    ├── instructions/
    └── skills/
```

---

## 🤝 Contributing

1. **Read `TODO.md`** before starting any work — phases must be completed in order
2. **Write failing tests first** (red), then implement (green)
3. **Run the full test suite** before opening a PR: `npm test`
4. **PRs require evidence**: paste test output + manual test screenshots in the PR description
5. **Small, focused PRs** — one feature or fix per PR
6. **Never remove tests** unless the feature is intentionally removed
7. Update `CLAUDE.md` / `AGENTS.md` with anything non-obvious you discovered

---

## 📜 License

MIT
