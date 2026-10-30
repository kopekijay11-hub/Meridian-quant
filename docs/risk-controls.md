# Risk controls

Risk in this system is not a policy document — it is code that runs on every cycle, plus operational controls that assume components will fail and plan for it.

## Circuit breakers

Three hard breakers guard the book, evaluated continuously by the tracker:

| Breaker | Code default | Ratified effective level |
|---|---|---|
| Peak-equity drawdown | 15% | **25%** |
| Daily loss | 5% | **10%** |
| Consecutive losses | 5 | **8** |

The effective levels are an operator decision, formally ratified after an audit caught documentation drift between the code defaults and the values the running system enforced. The record now states both numbers and which one governs — because a risk document that disagrees with the running code is itself a risk.

## Per-trade and portfolio controls

- **Per-trade risk caps** and portfolio-level exposure limits apply to every order, whatever signal produced it.
- **Position sizing is tiered by evidence quality.** A configuration's registry verdict and tier set its size multiplier; probe seats run at reduced size (fractions such as 0.3x–0.5x) while they accumulate forward evidence. Unmeasured configurations are never seated — there is no path by which an untested parameter set reaches an order.
- **An independent account guard** on the compute side cross-checks the account the bridge reports before orders flow.

## News risk — two layers, never conflated

1. **Static blackout calendar.** Scheduled high-impact events (central-bank decisions and similar) impose blackout windows wired into every engine — hourly, 4-hour, and intraday alike. A write-guard protects the calendar itself from silent corruption.
2. **Live headline shock.** Unscheduled shocks are a separate mechanism: a reduced-size mode rather than an edit to the static calendar. The two layers are deliberately kept distinct, because a calendar that absorbs ad-hoc entries stops being a calendar.

## The seating protocol

No arm joins or leaves the book informally. A seat change is: a registry edit, committed; a gated restart; and verification that quotes the startup book manifest — the loaded arms and explained skips must match the registry exactly, with the registry fingerprint unchanged across any restore (restores are byte-exact, by drill-learned rule). If it isn't in the manifest, it isn't seated, whatever the configuration files say.

## Operational risk controls

- **Supervision in both directions** — watchdog over tracker, tracker over watchdog, Guardian over bridge — each leg drill-proven (see [architecture.md](architecture.md)).
- **Fail-closed authentication.** The bridge token lives in one gitignored file; a missing token fails the import, never silently disables auth.
- **Secrets hygiene.** A tracked-secrets finding in audit was remediated by untracking and rotating every live credential, verified against the running system afterward. A secrets scan gates any merge of accumulated work into main.
- **Structured records.** Every live path emits timestamped JSON events; the trial ledger, alert log, heartbeat, and order book are machine-readable, so any claim about the book can be checked against a record rather than a recollection.
- **Demo-only status.** The book runs on a demo account. Promotion to live capital is a governance decision gated on clean out-of-sample evidence — it is not a deployment step, and it has not been taken.
