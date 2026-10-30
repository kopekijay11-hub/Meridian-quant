# Workflow example — the life of one hypothesis

This walkthrough shows how a single research idea travels the OrbitAlgo pipeline, from a paragraph in a register to a seating decision. It is a **methodology** example: the hypothesis below is illustrative and simplified, and no strategy logic, parameters, or configuration are shown — those are private (see the boundary in the [README](../README.md)).

## 1. Registration

An idea enters the **hypothesis register** with a mechanism paragraph — why the pattern should exist, in market terms. "Gold drifts with the handover between Asian and Western trading sessions" is a mechanism-shaped claim: it names who is trading, when, and why their flow might be predictable. An idea with no plausible mechanism is not registered, however good its backtest looks. Ideas are cheap; the register exists so their *count* is never forgotten.

## 2. Pre-registration

Before any data is touched, the programme commits a pre-registration document: the hypothesis, its instrument set, its timeframe, its expected trade frequency, and its **SELECT / VALIDATE / HOLDOUT date boundaries** — assigned from slices not already spent on it. From this commit onward, the funnel is fixed: no adding, removing, or re-slicing hypotheses mid-programme without an amendment commit that predates any affected run.

## 3. SELECT — the wide screen

The idea is screened across its instruments on the SELECT slice. Every parameter variant is a **charged trial** in the ledger. The output is a plain table per symbol: trades, net return, profit factor, t-statistic. Most ideas die here, quietly and cheaply — which is the screen doing its job.

## 4. VALIDATE — the gate

Survivors face the pre-declared bar on untouched data: **≥ 30 trades, profit factor ≥ 1.10, net return > 0**. Fall short on any leg and the verdict is recorded — FAIL, or FAIL-FLOOR for the trade-count leg — with full statistics, permanently. A recorded fail is a deliverable: it is a question the programme no longer has to ask.

## 5. HOLDOUT — one read

An advancing candidate gets exactly one read on its sealed holdout slice, against a **Šidák-corrected significance bar computed from the programme's total holdout spends**. Clear it and the verdict is PASS; miss it and the candidate joins the failure archive with its numbers intact. Either way the slice is spent — the same holdout is never read twice, and a tweaked candidate is a new hypothesis that must earn new slices.

## 6. The board

The programme publishes one board: every hypothesis × symbol with its furthest stage, verdict, and headline statistics — **including every reject, with its reason**. A board that hides its fails is treated as a failed board, because the board's credibility is the product.

## 7. The seating decision

A PASS makes a candidate *eligible for a decision* — nothing more. If the operator seats it, the seating protocol applies: registry edit and commit, gated restart, and verification against the startup book manifest, at probe size, labelled with the verdict it earned. Promotion beyond probe requires the forward evidence the demo book then accumulates.

That is the whole loop: **mechanism first, slices assigned before results, every try charged, one holdout read, rejects published, and a human making the final call.**
