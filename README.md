# AtmosAlpha
### Atmospheric Alpha — Event-Market Research & Systems Engineering

> A public technical portfolio for an independent quantitative systems project: event-driven market data, reproducible research, deterministic replay, operational observability, and risk-aware software design.

## Overview

**AtmosAlpha**—short for **Atmospheric Alpha**—is an independent quantitative systems project focused on the engineering foundations required for robust event-market research and event-driven software.

This public repository is a curated, non-operational technical portfolio. It contains safe architecture documentation and synthetic-example scaffolding intended to demonstrate engineering approach and system design.

> **Important:** This repository intentionally excludes proprietary research, production execution logic, strategy parameters, live venue integrations, account information, credentials, private infrastructure, real datasets, and performance-sensitive results.

## Focus Areas

- Event-stream ingestion, validation, normalization, and append-only capture
- Snapshot-plus-delta state reconstruction
- Reproducible historical research and deterministic replay
- Analytical workflows using Parquet and DuckDB
- Python and C++ systems engineering
- Monitoring, data-quality checks, and operational observability
- Risk-aware software design with safe defaults and explicit controls

## Reference Architecture

```text
Event Data Sources
        |
        v
Validation and Normalization
        |
        +-------------------+
        v                   v
Append-Only Storage   State Reconstruction
Parquet / DuckDB      Snapshot + Deltas
        |                   |
        v                   v
Deterministic Replay  Risk and Safety Layer
        |                   |
        +---------+---------+
                  v
       Metrics, Audits, and Observability
```

The public examples reflect the system-design principles behind the wider private project without exposing deployment topology, venue integrations, operating configuration, or proprietary methodology.

## Engineering Principles

- **Reproducibility:** Identical ordered inputs should produce identical replay results.
- **Separation of concerns:** Capture, storage, state reconstruction, research, monitoring, risk, and any authorization boundary should remain distinct.
- **Safe failure:** Stale, malformed, missing, or out-of-order data should lead to a safe state rather than unintended action.
- **Observability:** System health, data quality, ordering, and processing behavior should be measurable.
- **IP discipline:** Public materials should demonstrate capability without disclosing proprietary logic or operational details.

## Public Examples

| Example | Purpose |
|---|---|
| `examples/synthetic_event_stream/` | Synthetic event ingestion, validation, ordering, and append-only capture patterns |
| `examples/order_book_reconstruction/` | General snapshot-plus-delta state reconstruction concepts using fictional data |
| `examples/risk_limit_demo/` | Generic position limits, stale-data checks, safe failure, and audit-event patterns |
| `data/synthetic/` | Publicly safe, generated data only; no real market or production data |

The examples are intentionally non-operational. They are not connected to any live venue and do not contain order routing, trading signals, production configuration, or account functionality.

## Research and Validation

AtmosAlpha follows a staged workflow:

```text
Capture data -> Validate inputs -> Build reproducible datasets
-> Develop hypotheses -> Separate development and evaluation
-> Replay deterministically -> Measure robustness and failure modes
-> Require explicit safety and authorization boundaries
```

The public documentation explains this framework at a high level. Proprietary strategies, parameters, data, results, and implementation details remain private.

## Technology Focus

Python · C++ · Linux · DuckDB · Parquet · structured logging · deterministic replay · automated testing · GitHub Actions · cloud systems concepts

## Scope and Status

**Active independent project.**

This repository is a public technical portfolio and educational reference for safe, reproducible event-driven systems design. It is not a live trading product, a venue integration, financial advice, or an open-source release of the private AtmosAlpha platform.

## Professional Contact

Maintained by **H. Harper**.

- GitHub: [@FletchEm31](https://github.com/FletchEm31)
- LinkedIn: [Hayden Harper](https://www.linkedin.com/in/hayden-harper/)

For professional inquiries involving quantitative research, market data, data engineering, systems reliability, cloud infrastructure, Linux, or performance-oriented software, please connect through GitHub or LinkedIn.

## Copyright and Use

Copyright © 2026 H. Harper. All rights reserved.

This is a public technical portfolio, not an open-source release of the private AtmosAlpha platform. No license is granted to copy, modify, redistribute, commercially use, or create derivative works from this repository unless explicitly stated in writing.

See [NOTICE.md](NOTICE.md) for details.

## Disclaimer

Nothing in this repository constitutes investment advice, trading advice, financial advice, an offer to buy or sell any product, or a recommendation to participate in any market. Public examples are synthetic and provided solely for educational and portfolio purposes.