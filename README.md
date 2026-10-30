# OrbitAlgo — Autonomous Algorithmic Trading System

**An institutional-grade, multi-strategy trading platform — engineered, audited, and validated under a sealed out-of-sample protocol.**

> This repository is a **showcase**. The production codebase is private; what is documented here is the engineering: the architecture, the validation discipline, and the verified results. No strategy code, configuration, or credentials are published.

---

## Public vs private

This repository is the public artifact of a larger private system. The boundary is deliberate:

**Public (this repository):** architecture and system design · validation methodology and philosophy · the demo book's composition · audited engineering results and summaries · the workflow a research hypothesis follows from registration to a seating decision.

**Private (never published):** the production implementation and live execution code · trade configuration and risk-engine internals · the MetaTrader 5 bridge implementation · telemetry and operational data · proprietary strategy logic.

The point of the boundary is not secrecy for its own sake — it is that the engineering discipline is the portfolio piece, and it can be documented honestly and completely without exposing the system it governs.

## Project at a glance

| | |
|---|---|
| **System type** | Autonomous multi-strategy algorithmic trading platform |
| **Scope** | 24 registered symbols — precious metals, energy, equity indices, major & Canadian-dollar forex — on hourly and 4-hour timeframes |
| **Main components** | Strategy engines (ten families) · walk-forward research pipeline · market-data accessor over a checksummed corpus · ML layer (observer + shadow specialists) · risk layer with hard circuit breakers · MetaTrader 5 execution bridge · supervision stack (watchdog, bridge guardian, dashboard) |
| **Runtime environment** | Python on Linux (WSL2) bridged to a Windows-hosted MetaTrader 5 terminal; systemd-supervised services |
| **Validation methodology** | Sealed SELECT → VALIDATE → HOLDOUT protocol; every trial charged to an auditable ledger; family-wise (Šidák) significance correction before any result is called evidence |
| **Risk controls** | Hard breakers on drawdown, daily loss, and consecutive losses; per-trade and portfolio caps; news-event blackout; position sizing tiered by evidence quality |
| **Status** | Running autonomously on a MetaTrader 5 **demo account**; promotion to live capital is gated on clean out-of-sample evidence, not on a schedule |

## How to read this repository

- **README.md** (this file) — the overview: what the system is, what it has proven, and the results it stands on.
- **[docs/architecture.md](docs/architecture.md)** — system design: components, data flow, the supervision stack, and how the desk recovers from failure.
- **[docs/validation.md](docs/validation.md)** — the sealed validation protocol, the trial ledger, and what a PASS verdict does and does not mean.
- **[docs/risk-controls.md](docs/risk-controls.md)** — risk as code: breakers, blackout layers, sizing tiers, and the operational controls around the book.
- **[docs/audit-summary.md](docs/audit-summary.md)** — the independent audit record and how its findings were dispositioned.
- **[examples/workflow-example.md](examples/workflow-example.md)** — a walkthrough of one research hypothesis travelling the full pipeline, from registration to a seating decision.

---

## What it is

A fully autonomous trading system that researches, validates, and executes strategies across **24 registered symbols** — precious metals (gold, silver, platinum, palladium, copper), energy (crude and refined products), equity indices (German and Japanese benchmarks), and major & Canadian-dollar forex pairs — on hourly and 4-hour timeframes, quoted end-to-end in the broker's native instrument symbols via MetaTrader 5. The execution bridge is broker-agnostic by design: any MetaTrader 5 terminal can sit on the execution side.

The system runs end-to-end without human intervention: market data ingestion, signal generation across **ten strategy families**, an ML specialist layer, risk management with hard circuit breakers, order execution through a MetaTrader 5 bridge, a live monitoring dashboard, and automated alerting. It currently operates on a demo account by design: promotion to live capital is gated on clean out-of-sample evidence, under governance the system was specifically built to enforce. That discipline is the point of the engineering.

## Engineering highlights

- **40,000+ lines of Python**, one cohesive system: strategy engines, a walk-forward tuning pipeline, a bridge service connecting Linux (WSL2) to a Windows-hosted MetaTrader 5 terminal, a real-time dashboard, and a watchdog supervisor.
- **1,600+ automated tests**, with CI fully green on the current build — the full pytest suite, the strict type gate, and the coverage floor (68.02% against a 65% floor) all passing. The type-checking budget is enforced in CI as a ratchet: when CI revealed the true type-error count was higher than local runs had measured, the difference was root-caused and the debt was paid down to the original ceiling, file by file. The ceiling was never raised.
- **A single shared market-data layer, proven equivalent before it went live.** All market data — a stored corpus of 234 files spanning 39 symbols × 6 timeframes (~551 MiB, 2018–present) — is now read through one accessor backed by a sha256 checksum manifest. The repoint onto it was behavior-preserving by construction and by proof: output hashes captured before the change match the new layer's output exactly, across a 148-capture equivalence sweep of every served symbol, with zero discrepancies.
- **A book that announces itself.** At every startup the engines publish a machine-readable manifest of exactly which strategy arms loaded and which registry rows were skipped, with the gate that stopped each one named. A watchdog check compares the live book against the registry continuously and raises a critical alert on any unexplained absence — a detection, logging, and delivery chain proven end-to-end by live drills, including a deliberately planted phantom registry row.
- **Repeated independent full-system audits** (October 2026) spanning the execution bridge, risk layer, strategy engines, research pipeline, data layer, ML layers, and repository hygiene — the latest, graded October 29, 2026 under an expanded rubric that added runtime-activation evidence and the structured-logging standard, stands at **82/100 (B)**, with every material finding remediated or formally dispositioned and the corrections graded by the operator rather than self-certified.
- **Sealed validation protocol** — SELECT → VALIDATE → HOLDOUT with fail-closed governance: a holdout segment is spent exactly once, every tuning trial is written to an auditable ledger (**100,846 charged runs, 31,667 unique trials**), and family-wise significance is charged (Šidák correction) before any result may be called evidence.
- **A full independent audit cycle** (four phases) that found the original out-of-sample numbers were contaminated by selection leakage — then rebuilt the measurement pipeline so the leak is structurally impossible, and re-measured the entire book before trusting a single figure.
- **Supervision and hardening, second generation.** The desk runs as systemd user services with linger enabled (tracker, watchdog, dashboard), each with supervised restarts; the tracker and watchdog now also supervise *each other* — a dedicated thread in the tracker watches the watchdog's heartbeat, a failure mode found by drill, fixed, and then proven by a second drill in which the watchdog was deliberately stopped and the tracker's alert fired and cleared as designed. On the Windows side, a dedicated Bridge Guardian scheduled task keeps the MetaTrader 5 bridge alive, relaunching it within 10 seconds of a crash — a recovery proven by a live kill drill — and the terminal itself now autostarts at logon. Bridge authentication was rebuilt so the token lives only in its own gitignored file, with health verified by fingerprint, and later hardened to fail closed at import rather than run unauthenticated. A startup trap that re-archived trade history on every launch was found, fixed with a single-pass batch rewrite, and proven cured by a clean cold start after a full Windows reboot; Windows auto-restart has been disarmed, with a documented recovery path in its place. Every live path now emits structured, timestamped JSON logs under a single standard.
- **Repository discipline.** Eleven stacked feature branches — a month of audited work — were merged into main in ancestry order behind a secrets scan and per-branch CI gates, so the main branch, the running desk, and the documented state are the same code, verified by commit hash.
- **Reliability and security engineering**: a tracker-freeze root-cause investigation traced intermittent hangs to watchdog restart logic interacting with blocking I/O; the fix (isolated worker, hard timeouts, fail-fast reads) shipped with proof tests. A configuration audit caught a subsystem that could have routed orders outside its mandate; it was disabled and verified — the trade journal proved zero orders had ever been placed through it. A later audit found a secrets file tracked in version control; it was untracked and every live credential in it was rotated, with the running system verified healthy afterward.
- **Risk as code**: hard breakers on peak-equity drawdown, daily loss, and consecutive losses; per-trade and portfolio risk caps; a news-event blackout calendar wired into every engine; position sizes tiered by evidence quality — with the effective breaker levels (25% peak-equity drawdown, 10% daily loss, 8 consecutive losses) formally ratified as an operator decision after an audit caught documentation drift on exactly those numbers.

## The live demo book

The system currently runs an **eleven-arm book autonomously on a MetaTrader 5 demo account** — every arm below was confirmed loaded and cycling by name in the live startup manifest at the most recent supervised restart (October 30, 2026), across a market-data feed covering 24 symbols with zero gaps:

- **H1** — GOLD range grid · SILVER trend · USDJPY mean reversion · HEATOIL trend · LSGOIL trend · USOIL breakout · PLATINUM trend
- **H4** — CADCHF trend · CADNOK trend · GER40 trend · JPN225 trend

Every seat comes from a configuration the sealed protocol has measured — no arm trades on an untested parameter set. Position sizes are tiered by evidence quality, and seats that exist to accumulate forward evidence are labelled as such in the registry and capped at reduced size.

## The validation story

When the clean protocol re-measured all 22 tradable symbols, the results reset the expectations set by the old, leaky pipeline — which is exactly what the rebuild was for. On the H1 book, GOLD trend and SILVER trend recorded pass-class verdicts on never-before-seen data (GOLD trend: +6.92% on its sealed holdout across 146 trades); in the H4 timeframe program, CADCHF validated outright and now trades as one of the book's highest-conviction seats. The book continues to run on demo while candidates accumulate forward evidence. Promotion under the project's governance requires clearing a multiple-comparisons significance charge as well as the protocol's own bar — no candidate gets promoted on a single measurement.

That is the discipline this project demonstrates:
...[truncated 4427 chars]