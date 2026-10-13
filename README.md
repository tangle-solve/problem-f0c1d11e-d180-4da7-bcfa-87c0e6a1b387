<!-- solve-problem:f0c1d11e-d180-4da7-bcfa-87c0e6a1b387 -->
# Security under abrupt hashpower migration

A [Tangle Solve problem](https://solve.tangle.tools/problems/f0c1d11e-d180-4da7-bcfa-87c0e6a1b387).

## What is the problem?

Formalize confirmation-time/security guarantees for proof-of-work blockchains when the adversary controls the *timing and magnitude of hashpower changes* relative to difficulty-retarget epochs.

Setting: Bitcoin-style backbone with variable difficulty. Honest network has hashpower that can jump discontinuously ("abrupt migration" — e.g., miners exit or arrive mid-epoch). Difficulty retargets every epoch of N blocks (N = 2016 in Bitcoin) based on timestamps. The adversary has total hashpower fraction ≤ 1/2 − ε *at all times* (or a time-varying bound), controls network delivery with delay Δ, and can manipulate timestamps within the consensus-allowed slack.

The goal: define and bound **common prefix / chain quality / confirmation soundness** — the probability that a block k-deep is later revoked — using only *conservatively observable quantities* (quantities a client can measure from the chain itself, e.g., observed block-arrival rates and timestamps, which may be adversary-influenced). The open question is a tight analysis through (a) retarget boundaries, where difficulty is set from possibly manipulated timestamps and stale hashrate, and (b) adversarial timing of hashpower shocks, e.g., a shock just after a retarget so difficulty is wrong for a full epoch.

Ambiguity: "abrupt hashpower migration" is not formally defined in the census record; the reasonable interpretation is a piecewise-constant honest hashrate with adversarially chosen jump times, subject to a global bound on instantaneous adversarial fraction. A solver should state their formalization explicitly.

## Why does it matter?

Confirmation depth recommendations (e.g., "wait 6 blocks") assume stationary mining; during real hashpower shocks (China's 2021 mining ban, hashrate collapses of ~50%), these heuristics have no proven meaning. A solution yields safe, self-certifying confirmation rules for wallets and exchanges during exactly the periods when double-spend attacks are most profitable — post-shock epochs with mis-set difficulty. It would also extend the variable-difficulty backbone analysis (which assumes slowly changing or bounded-variance difficulty) to realistic adversarial dynamics.

## What's already known?

- Garay–Kiayias–Leonardos (2017, ePrint 2016/1048) extend the Bitcoin backbone analysis to variable difficulty with a target-recalculation function satisfying bounded growth; they prove common prefix and chain quality when honest majority holds *per unit of target-adjusted work* and hashrate variation is smooth. Their model does not handle adversarially timed jumps or retarget-boundary transients.
- Guo–Ren (2022, arXiv:2203.06357) give tight latency–security bounds at *fixed* difficulty/hashing rate: the probability a k-deep block is revoked decays exponentially in k, with tight constants — but only in the stationary regime.
- Known primitives: the longest-chain-by-*work* (not length) rule is robust to difficulty changes; retarget functions are timestamp-manipulable only within bounded slack (e.g., MTP rules); difficulty cannot adjust more than a factor of 4 per epoch in Bitcoin.
- Gap: no analysis combines time-varying hashrate + timestamp manipulation + retarget boundaries + observable-quantity confirmation rules. Related tooling exists in change-point detection and nonstationary random walks, but has not been ported to this setting.

## What would count as solving it?

1. **Definition:** A formal model with piecewise-constant honest hashrate h(t), adversarial fraction ≤ 1/2 − ε, delay Δ, timestamp-slack parameter, retarget function R; plus a definition of "conservatively observable" chain-derived statistics.
2. **Theorem (upper bound):** For all adversarial migration schedules and timestamp strategies, the probability that a block with observable work-depth ≥ w is revoked is ≤ f(w, ε, Δ, observable quantities), with f explicit and exponentially decaying when observables confirm honest dominance.
3. **Lower bound:** A matching attack showing the bound is tight for some shock schedule (e.g., shock timed right after retarget).
4. **Algorithm:** A confirmation rule usable by a client that reads only the chain and outputs safe/unsafe, with proven soundness.

Partial results (e.g., only abrupt decreases, no timestamp manipulation, asymptotic-only bounds) are valuable but not full resolution.

## How to start

1. Read Garay–Kiayias–Leonardos 2017, Sections on the variable-difficulty model and the target-recalculation function — extract the exact per-epoch invariants (work accumulation rate, bounded target change) that break under abrupt jumps.
2. Read Guo–Ren 2022 for the stationary confirmation bound machinery (random-walk/recurrence analysis of private-attack lead) — the task is to redo this with a random walk whose drift is piecewise-constant and adversarially timed.
3. Decide the observability model first: what can a client infer about h(t) from block arrival times when timestamps lie? Natural approach: use conservative bounds (e.g., difficulty ≤ observed interarrival implies work rate ≤ ...) and define a "worst-case effective adversarial fraction" estimator; prove it upper-bounds the true fraction.
4. Main obstacle: at retarget boundaries, adversary-controlled timestamps couple future difficulty to past lying, creating feedback; consider first freezing the retarget function (idealized: perfect difficulty) to solve the pure migration problem, then adding bounded timestamp slack.

## Key references

- J. Garay, A. Kiayias, N. Leonardos. *The Bitcoin Backbone Protocol with Chains of Variable Difficulty.* IACR ePrint 2016/1048 (published 2017). Extends the backbone analysis to variable difficulty, proving common prefix/chain quality under bounded-rate target recalculation; the baseline this problem generalizes. https://eprint.iacr.org/2016/1048
- Y. Guo, F. Ren. *Bitcoin's Latency–Security Analysis Made Simple.* arXiv:2203.06357. Derives tight closed-form bounds on double-spend probability vs. confirmation depth at fixed hashrate/difficulty; supplies the stationary analysis to be extended. https://arxiv.org/abs/2203.06357
- (verify) For timestamp-manipulation constraints under retargeting, consensus rules in Bitcoin Core (median-time-past) bound how far timestamps may deviate; no canonical citation was provided in the census record.

## Warnings

- The census record is informal; "abrupt hashpower migration" and "conservatively observable quantities" must be pinned down before proving anything — different formalizations yield different (and differently hard) problems.
- Common mistake: analyzing chain *length* instead of cumulative *work*; under variable difficulty only the work metric is sound.
- Dead end: assuming block-arrival rates reveal honest hashrate directly — the adversary controls timestamps and can withhold blocks, so naive estimators are gamed; observability must be defined adversarially.
- Subtlety: Bitcoin's 4× retarget cap means a >4× hashrate drop leaves difficulty wrong for multiple epochs; bounds must cover sustained mismatch, not just one boundary.
- Verify the 2022 paper's exact model assumptions (they may assume synchronized/Δ-delay networks and no timestamp games) before importing lemmas.

---
*`BCGT-000023` · flagged for review*

## Acceptance guidance

1. **Definition:** A formal model with piecewise-constant honest hashrate h(t), adversarial fraction ≤ 1/2 − ε, delay Δ, timestamp-slack parameter, retarget function R; plus a definition of "conservatively observable" chain-derived statistics.
2. **Theorem (upper bound):** For all adversarial migration schedules and timestamp strategies, the probability that a block with observable work-depth ≥ w is revoked is ≤ f(w, ε, Δ, observable quantities), with f explicit and exponentially decaying when observables confirm honest dominance.
3. **Lower bound:** A matching attack showing the bound is tight for some shock schedule (e.g., shock timed right after retarget).
4. **Algorithm:** A confirmation rule usable by a client that reads only the chain and outputs safe/unsafe, with proven soundness.

Partial results (e.g., only abrupt decreases, no timestamp manipulation, asymptotic-only bounds) are valuable but not full resolution.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/f0c1d11e-d180-4da7-bcfa-87c0e6a1b387?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000023.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000023)
