# AtmosAlpha
### Atmospheric Alpha: event-market research and trading systems

> A public technical portfolio for an independent quantitative systems project: a research and execution platform for weather event contracts, built and run end to end by one engineer.
>
> **This repository is the public window.** The production platform is private. Nothing here contains trading logic, strategy parameters, results, venue integrations, deployment topology, credentials or real data.

---

## What AtmosAlpha is

AtmosAlpha trades and researches **event contracts on U.S. regulated exchanges**, mainly weather markets: daily high and low temperature across ten U.S. cities, hourly temperature, and daily rainfall. A second venue and an exchange-listed equivalent are recorded for comparison, and short-dated crypto event markets are recorded for research only.

The private platform covers the whole path:

```text
capture every market and weather feed  →  store it raw, then clean it  →  research and backtest on point-in-time data
→  promote a strategy through shadow, dry-run, demo and pilot stages  →  execute on a dedicated node with hard risk limits
→  monitor everything, back everything up, and document the system automatically
```

It started in May 2026 inside a private infrastructure project and is now in its third generation. It is designed, operated and maintained by one person, with AI coding assistants doing much of the implementation under written operating rules (see [How it's built](#how-its-built)).

## At a glance

| Area | What the private platform does |
|---|---|
| **Data capture** | Exchange WebSocket feeds 24/7, plus weather data: numerical forecast models (every ensemble member, archived as issued), official station observations every 20 seconds, model release times |
| **Storage** | About 20 DuckDB databases fed from raw JSONL; Parquet exports and archives |
| **Time** | Four timestamps on every market event: exchange, capture, relay and ingest |
| **Research** | 100 catalogued studies and backtests, **negative results kept** (34 so far) |
| **Validation** | A 12-test evidence checklist and an 8-stage promotion ladder from idea to live |
| **Execution** | C++ order engines on a dedicated execution node; the recording node doesn't send orders |
| **Monitoring** | 43 automated end-to-end checks every 2 minutes, alerts to a phone, self-repair for a short whitelist |
| **Operations** | Checksum-verified deploys with backups first; encrypted backups in two failure domains; a system overview that regenerates itself daily |

## Architecture

```text
 Event exchanges ──WebSocket──▶ ┌─────────────────────────────────────┐ ──orders──▶ exchange
                                │  EXECUTION NODE                      │
                                │  capture (RAM spool) · C++ engines   │
                                │  risk limits · kill switches         │
                                └──────────────┬──────────────────────┘
                                               │ encrypted relay (every frame, order, fill)
                                ┌──────────────▼──────────────────────┐
 Weather and model APIs ──────▶ │  RECORDING NODE (never sends orders) │
                                │  raw ▶ clean ▶ derived               │
                                │  dry-run + shadow strategies         │
                                │  monitoring · alerts · status API    │
                                └──────┬───────────────────┬──────────┘
                           data copies │                   │ encrypted backups
                                ┌──────▼───────────────────▼──────────┐
                                │  RESEARCH WORKSTATION                │
                                │  backtests · replay · terminal UI    │
                                │  encrypted vaults · data archive     │
                                └──────────────────────────────────────┘
```

**Roles never overlap.** By standing rule, live orders come from the execution node only. The recording node records everything and runs every paper strategy, but doesn't trade. Research runs on a separate workstation against copies of the data.

## Design principles

| Principle | What it means in practice |
|---|---|
| **One home per thing** | Every dataset, file type and record has exactly one authoritative location; everything else is a labelled copy, backup or cache |
| **Raw first, never edited** | Feeds land as raw records before any transform; transforms can always be re-run from raw; every event row keeps its original payload |
| **Point-in-time, always** | A backtest decision at time *T* may only use data the system had received before *T*. Forecasts come from as-issued archives, never later reanalysis |
| **Every result is reproducible** | Each study and backtest writes a run record: code commit, parameters, input checksums and its stage. No record, no evidence |
| **One source of truth for money** | Balances and P&L come from one module that reads the exchange; if it can't be read, the system says so and sends nothing |
| **Fail safe** | Stale, missing or out-of-order data leads to a safe state, never an unintended order |
| **No quiet failures** | Every job has a health check; a failed write is an alert, never a skipped log line |
| **Frozen names** | Production file, table and job names are never renamed; that makes them stable keys for documentation and lineage |
| **Two failure domains** | Irreplaceable data needs a copy outside the cloud account *and* outside the house |

### Data layers

| Layer | Holds | Rule |
|---|---|---|
| **raw** | Data exactly as received | Never edited; append-only |
| **clean** | Typed, validated, de-duplicated, conformed data | Rebuildable from raw at any time |
| **derived** | Features, model outputs, simulations | Rebuildable from clean plus code at a commit |
| **records** | Decisions, orders, fills, ledgers | Never rebuilt; corrections are new rows |
| **reference** | Stations, settlement rules, lookups | Hand-maintained, versioned |

The same idea as the medallion (bronze / silver / gold) architecture, with two extra categories for the data that can never be recreated.

## Research and validation

**The stage ladder.** A strategy moves up only with evidence, and every step up needs the owner's explicit go:

| # | Stage | What it proves |
|---|---|---|
| 1 | Alpha study | Is there a signal at all, against the market's own price? |
| 2 | Backtest | Does the frozen rule work on unseen, chronological data, after fees? |
| 3 | Replay | Does it survive real recorded order books and a conservative fill model? |
| 4 | Shadow | Does it behave the same on new live data (paper)? |
| 5 | Dry run | Does the real engine, real code path and real deploy work, with orders disabled? |
| 6 | Demo | Does order handling work against the exchange's test environment? |
| 7 | Pilot | Tiny real size, hard caps, kill switch armed |
| 8 | Live | Target size, full monitoring |

**The evidence checklist.** A result counts only after the tests that apply to it are done and recorded:

1. Data integrity and leakage audit
2. Signal test against a simple baseline
3. In-sample backtest (debugging only, never proof)
4. Out-of-sample test on unseen chronological data
5. Walk-forward validation
6. Execution-realistic backtest: fees, spread, latency, partial fills, queue position
7. Parameter sensitivity
8. Regime and subgroup analysis
9. Block bootstrap and Monte Carlo for drawdowns
10. Multiple-testing correction (every trial tracked, including the failures)
11. Ablation against simpler versions
12. Forward shadow testing before capital

**Point-in-time data classes.** Downloaded history is split by whether it can change. *Fixed events* (trades, settlements) are usable with a stated availability delay. *Revised data* (forecasts, corrected observations) is usable only with its issue times; otherwise it's treated as climatology, not as something the strategy could have known.

## Lessons that shaped the system

| What happened | What changed |
|---|---|
| A full disk on the recording node stopped ingestion for about 47 hours, and a receiver restarted roughly 27,000 times without anyone noticing | An end-to-end monitor that checks every stage of the pipeline every 2 minutes, a self-repair layer, and the "no quiet failures" rule |
| Database indexes on the largest append-only tables exhausted memory at around 54 million rows | Indexes removed from hot write paths; time-range scans rely on DuckDB's zone maps instead |
| Production runs copies of repository files from fixed paths on three machines, so a routine cleanup could silently break the next deploy | An automated audit of every file production depends on, run before anything is moved or renamed |
| A daily data job failed for three days before it was spotted | A new standard, being rolled out: every scheduled job gets its own health check, and each table a freshness target paired with a check that can actually verify it |

## How it's built

AtmosAlpha is a one-person project. The owner designs the architecture, sets the rules and makes every stage and risk decision; AI coding assistants write much of the code. That only works with structure around it:

- **Written operating rules** in the repository that every AI session loads: what it may do on its own (docs, tests, reversible fixes) and what it must ask first (strategy logic, cloud infrastructure, schedules, deleting data, anything near a live run).
- **Handoffs between sessions**: each work session ends with a task-list update and a structured handoff note, and the next session must confirm which rules it read before it starts.
- **Commit straight to `main`**, with tests, and deploys that are verified by checksum against the commit, with backups taken first.

## In this repository

| Path | What it is | Status |
|---|---|---|
| `docs/architecture-overview.md` | The public reference architecture | Available |
| `docs/research-and-validation.md` | The research framework at a high level | Available |
| `docs/risk-and-safety-design.md` | Risk and safety design principles | Available |
| `examples/synthetic_event_stream/` | Event validation and append-only capture, on generated data | Planned |
| `examples/order_book_reconstruction/` | Snapshot-plus-delta order book reconstruction, on fictional data | Planned |
| `examples/risk_limit_demo/` | Position limits, stale-data checks and safe failure | Planned |
| `data/synthetic/` | Generated data only; no real market or production data | Planned |

The examples are intentionally non-operational: no venue connection, order routing, trading signals, production configuration or account functionality.

## Technology

Python · C++ · Linux (Ubuntu) · DuckDB · Parquet · JSONL · WebSockets · WireGuard · systemd · AWS · GitHub Actions · LUKS and Cryptomator encryption · Textual (terminal UI) · pytest

## Scope and status

**Active independent project, private production platform.** This repository is a public portfolio and educational reference for safe, reproducible event-driven systems design. It is not a trading product, a venue integration, financial advice, or an open-source release of the private AtmosAlpha platform.

## Professional contact

Maintained by **H. Harper**.

- GitHub: [@FletchEm31](https://github.com/FletchEm31)
- LinkedIn: [Hayden Harper](https://www.linkedin.com/in/hayden-harper/)

For professional inquiries involving quantitative research, market data, data engineering, systems reliability, cloud infrastructure, Linux, or performance-oriented software, please connect through GitHub or LinkedIn.

## Copyright and use

Copyright © 2026 H. Harper. All rights reserved.

This is a public technical portfolio, not an open-source release of the private AtmosAlpha platform. No license is granted to copy, modify, redistribute, commercially use, or create derivative works from this repository unless explicitly stated in writing. See [NOTICE.md](NOTICE.md).

## Disclaimer

Nothing in this repository constitutes investment advice, trading advice, financial advice, an offer to buy or sell any product, or a recommendation to participate in any market. Public examples are synthetic and provided solely for educational and portfolio purposes.
