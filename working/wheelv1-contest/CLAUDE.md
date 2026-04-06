# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WheelV1 Contest Engine v3 — a FastAPI-based fortune wheel (raffle) system with mathematical profit guarantees. Implements a "Budget Envelope" algorithm ensuring `profit >= 0` on every spin, with an integral controller driving margin toward a configurable target.

**Stack:** Python 3.12, FastAPI, Pydantic 2, SQLite (WAL mode), pytest.

## Commands

```bash
# Run the server
uvicorn main:app --host 0.0.0.0 --port 8000

# Run all tests
pytest test_engine.py test_api.py test_database.py test_ui.py -v

# Run a single test file
pytest test_engine.py -v

# Run a single test
pytest test_engine.py::TestBudgetEnvelope::test_profit_never_negative -v

# Run simulation CLI
python simulate.py --spins 10000 --margin 0.25 --seed 42

# Docker
docker compose up --build
```

## Architecture

```
main.py  →  FastAPI routes, HTTP Basic Auth, lifespan
              ↓
database.py  →  WheelManager: multi-wheel orchestration, in-memory engine cache,
                 SQLite persistence (9 tables), batched commits (every 500 spins)
              ↓
engine.py  →  SpinEngine: core algorithm per wheel
              - Budget Envelope: prize awarded only if cost <= (revenue × (1-min_margin) - total_payout)
              - Integral Controller: adjusts win-rate to converge actual margin → target margin
              - Tier system: bronze/silver/gold/web with per-tier win rates
              - Jackpot: contribution from profit, chance-based hit, configurable payout
              ↓
models.py  →  All Pydantic DTOs: configs, requests, responses, UI models
```

### Key algorithmic invariant
Every spin enforces `budget = revenue × (1 - min_margin) - total_payout`. A prize is only awarded if `prize.cost <= budget`. This guarantees the operator never loses money.

### Engine cache pattern
`WheelManager` holds a `dict[int, SpinEngine]` cache. Engines are lazily loaded from SQLite on first access. State syncs incrementally (dirty players/prizes/tiers/rounds only), with full commits every 500 spins.

### Tier system
Players are assigned tiers by cumulative spend: Bronze (0-30), Silver (30-100), Gold (100+). Web-credit players get a separate "web" tier with reduced win rate (base × 0.2) and reduced payout (cost/5).

## Environment Variables

| Variable | Default | Purpose |
|---|---|---|
| `WHEELV1_DB_PATH` | `wheelv1.db` | SQLite database file path |
| `WHEELV1_ADMIN_USER` | `admin` | HTTP Basic Auth username |
| `WHEELV1_ADMIN_PASSWORD` | *(empty = auth disabled)* | HTTP Basic Auth password |

## API Structure

39 endpoints across categories: **ui** (player-facing, no auth), **wheels** CRUD, **prizes**, **game** (spin/round), **stats**, **management**. UI endpoints are prefixed `/wheels/{id}/ui/`. Admin endpoints require Basic Auth.

## Test Suites

- `test_engine.py` — Algorithm guarantees: budget envelope, margin convergence, tiers, jackpot, prize limits, state export/import
- `test_api.py` — FastAPI endpoint integration tests (TestClient)
- `test_database.py` — WheelManager persistence and SQLite round-trip
- `test_ui.py` — UI contract validation (segment mapping, history pagination)

Each test creates a fresh temporary SQLite DB via fixtures.

## Code Conventions

- Type hints throughout (Python 3.10+ style)
- Comments and README are in Russian
- snake_case for functions/variables, CamelCase for classes
- No formal linter config — code generally follows PEP 8
- `__slots__` used on internal state classes (`_Player`, `_RoundState`) for memory efficiency
