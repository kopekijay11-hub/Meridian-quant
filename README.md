# OrbitAlgo — Autonomous Algorithmic Trading System

**An institutional-grade, multi-strategy trading platform — engineered, audited, and validated under a sealed out-of-sample protocol.**

> This repository is a **showcase**. The production codebase is private; what is documented here is the engineering: the architecture, the validation discipline, and the verified results. No strategy code, configuration, or credentials are published.

---

## What it is

A fully autonomous trading system that researches, validates, and executes strategies across **24 registered symbols** — precious metals (gold, silver, platinum, palladium, copper), energy (crude and refined products), equity indices (German and Japanese benchmarks), and major & Canadian-dollar forex pairs — on hourly and 4-hour timeframes, quoted end-to-end in the broker's native instrument symbols via MetaTrader 5. The execution bridge is broker-agnostic by design: any MetaTrader 5 terminal can sit on the execution side.

The system runs end-to-end without human intervention: market data ingestion, signal generation across **ten strategy families**, an ML specialist layer, risk management with hard circuit breakers, order execution through a MetaTrader 5 bridge, a live monitoring dashboard, and automated alerting. It currently operates on a demo account by design: promotion to live capital is gated on clean out-of-sample evidence, under governance the system was specifically built to enforce. That discipline is the point of the engineering.

## Engineering highlights

- **40,000+ lines of Python**, one cohesive system: strategy engines, a walk-forward tuning pipeline, a bridge service connecting Linux (WSL2) to a Windows-hosted MetaTrader 5 terminal, a real-time dashboard, and a watchdog supervisor.
- **1,600+ automated tests**, with CI fully green on the current build — the full pytest suite, the strict type gate, and the coverage floor (68.02% against a 65% floor) all passing. The type-checking budget is enforced in CI as a ratchet: when CI revealed the true type-error count was higher than local runs had measured, the difference was root-caused and the debt was paid down to the original ceiling, file by file. The ceiling was never raised.
- **A single shared market-data layer, proven equivalent before it went live.** All market data — a stored corpus of 234 files spanning 39 symbols × 6 timeframes (~551 MiB, 2018–present) — is now read through one accessor backed by a sha256 checksum manifest. The repoint onto it was behavior-preserving by construction and by proof: output hashes captured before the change match the new layer's output exactly, across a 148-capture equivalence sweep of every served symbol, with zero discrepancies.
- **Repeated independent full-system audits** (October 2026) spanning the execution bridge, risk layer, strategy engines, research pipeline, data layer, ML layers, and repository hygiene — the latest, graded October 29, 2026 under an expanded rubric that added runtime-activation evidence and the structured-logging standard, stands at **82/100 (B)**, with every material finding remediated or formally dispositioned and the corrections graded by the operator rather than self-certified.
- **Sealed validation protocol** — SELECT → VALIDATE → HOLDOUT with fail-closed governance: a holdout segment is spent exactly once, every tuning trial is written to an auditable ledger (**100,846 charged runs, 31,667 unique trials**), and family-wise significance is charged (Šidák correction) before any result may be called evidence.
- **A full independent audit cycle** (four phases) that found the original out-of-sample numbers were contaminated by selection leakage — then rebuilt the measurement pipeline so the leak is structurally impossible, and re-measured the entire book before trusting a single figure.
- **Supervision and hardening, second generation.** The desk runs as systemd user services with linger enabled (tracker, watchdog, dashboard), each with supervised restarts; on the Windows side, a dedicated Bridge Guardian scheduled task keeps the MetaTrader 5 bridge alive, relaunching it within 10 seconds of a crash — a recovery proven by a live kill drill. Bridge authentication was rebuilt so the token lives only in its own gitignored file, with health verified by fingerprint. A startup trap that re-archived trade history on every launch was found, fixed with a single-pass batch rewrite, and proven cured by a clean cold start after a full Windows reboot; Windows auto-restart has been disarmed, with a documented recovery path in its place. Every live path now emits structured, timestamped JSON logs under a single standard.
- **Reliability and security engineering**: a tracker-freeze root-cause investigation traced intermittent hangs to watchdog restart logic interacting with blocking I/O; the fix (isolated worker, hard timeouts, fail-fast reads) shipped with proof tests. A configuration audit caught a subsystem that could have routed orders outside its mandate; it was disabled and verified — the trade journal proved zero orders had ever been placed through it. A later audit found a secrets file tracked in version control; it was untracked and every live credential in it was rotated, with the running system verified healthy afterward.
- **Risk as code**: hard breakers on peak-equity drawdown, daily loss, and consecutive losses; per-trade and portfolio risk caps; a news-event blackout calendar wired into every engine; position sizes tiered by evidence quality — with the effective breaker levels (25% peak-equity drawdown, 10% daily loss, 8 consecutive losses) formally ratified as an operator decision after an audit caught documentation drift on exactly those numbers.

## The live demo book

The system currently runs an **eleven-arm book autonomously on a MetaTrader 5 demo account** — every arm below was confirmed loaded and cycling by name in the live logs at the most recent supervised restart (October 29, 2026), across a market-data feed covering 24 symbols with zero gaps:

- **H1** — GOLD range grid · SILVER trend · USDJPY mean reversion · HEATOIL trend · LSGOIL trend · USOIL breakout · PLATINUM trend
- **H4** — CADCHF trend · CADNOK trend · GER40 trend · JPN225 trend

Every seat comes from a configuration the sealed protocol has measured — no arm trades on an untested parameter set. Position sizes are tiered by evidence quality, and seats that exist to accumulate forward evidence are labelled as such in the registry and capped at reduced size.

## The validation story

When the clean protocol re-measured all 22 tradable symbols, the results reset the expectations set by the old, leaky pipeline — which is exactly what the rebuild was for. On the H1 book, GOLD trend and SILVER trend recorded pass-class verdicts on never-before-seen data (GOLD trend: +6.92% on its sealed holdout across 146 trades); in the H4 timeframe program, CADCHF validated outright and now trades as one of the book's highest-conviction seats. The book continues to run on demo while candidates accumulate forward evidence. Promotion under the project's governance requires clearing a multiple-comparisons significance charge as well as the protocol's own bar — no candidate gets promoted on a single measurement.

That is the discipline this project demonstrates: measurement infrastructure rigorous enough to be trusted when it says *yes*.

## The full package — front end to back end

This is a full-stack system, designed and built end-to-end by one engineer:

- **Front end** — a real-time operations dashboard (NiceGUI web UI): live positions, equity, per-strategy-arm state, risk-breaker status, and alerting, served locally by design. It is the operator's window into everything below it.
- **Middle layer** — an execution bridge API (FastAPI, token-authenticated REST with idempotent order endpoints) connecting the Linux compute environment (WSL2) to a Windows-hosted MetaTrader 5 terminal (demo account), with an independent account-guard check on the compute side.
- **Back end** — the strategy engines, the walk-forward research pipeline with its schema-versioned trial ledger, the risk layer with hard circuit breakers, a SQLite trade journal, and a heartbeat-monitored watchdog supervisor.

## Architecture at a glance

- **Strategy engines** (H1 primary; H4 book; intraday series) hosting ten families — trend, breakout, mean-reversion, range grid, session breakout, liquidity sweep, Donchian, swing, and statistical arbitrage
- **Market-data layer**: one shared accessor over a checksummed corpus (234 files, 39 symbols × 6 timeframes, ~551 MiB, 2018–present) with a sha256 manifest — every consumer reads through the same door, and the layer's equivalence to the parsers it replaced is hash-proven
- **Walk-forward tuner** with a sealed segment pipeline and a schema-versioned trial ledger
- **ML layer** (gradient-boosted models): an expected-R model (LightGBM) serves in an observer role informing position sizing, and a specialist layer — one bundle set per strategy family — runs in permanent shadow mode after a dedicated audit of its training-data provenance, per-family expectancy, and sizing impact, scoring and logging only while it accumulates the evidence base that will govern any future retrain decision
- **Execution bridge** between the Linux compute environment and MetaTrader 5, with token authentication held in a dedicated gitignored file, fingerprint-verified health checks, and an independent account-guard check on the compute side
- **Supervision**: systemd user services with linger (tracker, watchdog, dashboard) under supervised restarts, a Windows Bridge Guardian that relaunches the MT5 bridge within 10 seconds of a crash, heartbeat monitoring with failure classification, bot-based alerting, and a local operations dashboard
- **Research stack** (isolated environment): indicator-parity certification against TA-Lib, vectorized-backtest reconciliation, and seeded, replayable hyperparameter studies

## Skills demonstrated

Python (large-system design) · full-stack delivery (web dashboard UI · REST API bridge · execution engines · data layer) · pytest at scale · type-safe engineering (mypy ratchet) · CI/CD · statistical validation & multiple-testing discipline · walk-forward / out-of-sample methodology · API & bridge integration (MetaTrader 5) · Linux/WSL systems work · systemd service supervision · incident root-cause analysis · technical documentation and audit reporting

---

*Built and operated by one self-taught engineer. The private repository contains the full implementation, the complete audit trail, and the trial ledger; it is available for review in the context of employment or contract discussions.*
