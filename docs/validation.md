# Validation

The system's core claim is not that it trades — it is that its measurements can be trusted. This document describes the protocol that makes a result evidence, and what a verdict does and does not entitle anyone to conclude.

## The sealed protocol

Every candidate strategy configuration travels the same three segments, in order, fail-closed:

1. **SELECT** — a wide screen over a defined historical slice. Variants are cheap here and numerous; every one is charged.
2. **VALIDATE** — survivors are measured on a separate slice they were not selected on. Advancement requires a pre-declared bar: at least 30 trades (below that floor the verdict is FAIL-FLOOR regardless of return), profit factor ≥ 1.10, and net return > 0.
3. **HOLDOUT** — each advancing candidate receives exactly **one** read on a sealed segment. The segment is then spent: never re-read, never re-tuned against, for that candidate, ever.

A slice spent on a symbol or family stays spent. A modified candidate is a new hypothesis: it needs new slices and a new charge. There is no mechanism in the pipeline for quietly looking twice — the earlier version of the system had exactly that flaw (selection leakage in its tuning pipeline), an independent audit found it, and the pipeline was rebuilt so the leak is structurally impossible rather than procedurally discouraged.

## The trial ledger

Every measurement run — every variant, every screen, every re-tune — is written to a schema-versioned ledger before its result is accepted. Current census: **100,846 charged runs, 31,667 unique trials.** The ledger is the programme's memory and its honesty mechanism: a winner selected from ten thousand charged trials is a different claim from the same winner selected from ten, and the ledger is what makes the count undeniable.

Failed candidates are kept with the same care as winners. The failure archive is permanent and first-class — knowing precisely which ideas were tested and rejected, on what data, is what separates a measured book from a lucky one.

## Significance is charged

A result clears the bar only after a family-wise significance correction (Šidák), computed from the total number of holdout spends in its programme — the bar rises with the number of tries. In the October 2026 clean-board re-measurement (22 symbols, four new holdout spends, bar |t| > 2.49), **zero of four candidates cleared the corrected bar, and accordingly none was promoted** — including candidates with strongly positive raw holdout returns. Running on the demo book and being promoted are different states, and the protocol keeps them different on purpose.

## What a verdict means

- **PASS** — the candidate cleared its gates. It is *eligible for an operator seating decision* at probe scale under the seating protocol (a registry edit, a commit, a gated restart, and verification against the startup manifest). PASS is not a promotion, a promise, or a profitability claim.
- **FAIL / FAIL-FLOOR** — recorded with full statistics and kept permanently. A fail is a deliverable: it is a hypothesis the programme no longer has to wonder about.
- **Running ≠ validated.** Under the keep-all decision, measured arms continue to run on the demo book at tiered sizes while accumulating forward evidence, each labelled in the registry with the verdict it actually earned.

## Programmes follow the same law

New research enters through a hypothesis register: a mechanism paragraph (why the pattern should exist) is written *before* any run, instruments and slices are assigned in advance, and pre-registration is committed before screening begins. A screening programme currently in flight covers seven pre-registered market-pattern hypotheses under exactly these rules; any system-generated strategy candidates in future work are named before testing, charged at generation, and archived permanently whether they pass or fail.
