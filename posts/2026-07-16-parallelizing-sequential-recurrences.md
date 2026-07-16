# Parallelizing the sequential: recurrence shapes, and a trade chain on a GPU

*Published July 16, 2026. The events: March–June 2026. Provenance stated per [this journal's new standard](2026-07-16-auditing-my-own-origin-stories.md): the problem and the acceptance criteria are mine; the algorithms came from Claude's training, not from me and not from the web (the audit checked — zero relevant lookups in the corpus). My contribution is the doctrine, the routing, and the oracles that keep it honest.*

Backtesting has a serialization problem wearing two costumes. Stateful indicators are recurrences — today's value depends on yesterday's — and a single-position trade chain is worse: you can't know when trade N+1 begins until you know when trade N exits. Both look inherently sequential. Both ended up parallel or fast anyway, and the interesting part is *how the decision gets made*, because the answer is not "put everything on the GPU."

## Recurrence shapes: route by the math, not by enthusiasm

Every stateful indicator in the engine is classified by the shape of its recurrence, and each shape routes to the backend that matches it:

- **Linear IIR** (EMA-family — the recurrence is a linear filter): `scipy.lfilter`. A linear recurrence *is* a filter; solved problem, no GPU needed.
- **Non-linear state machine** (Wilder-style smoothing chains, QQE, STC — branching on state): Numba `@njit`, **deliberately forced onto the CPU even when the rest of the run is on GPU.** A tight compiled sequential loop with one host round-trip beats a launch storm — the profile that killed the GPU version was on the order of 763k kernel launches. Launches dominate; the math never got a vote.
- **Path-dependent ratchet** (monotone running extremes): parallel prefix scan — the associative-operator trick that turns a "sequential" running max into a log-depth tree.
- **Rolling window**: `filter1d`-style convolution. Not a recurrence at all once you look at it straight.

None of these primitives is mine. Linear-recurrence-as-scan is literature (it's the trick under Mamba-era sequence models); Ehlers spent a career writing indicators as filters; Numba is Numba. An adversarial prior-art search also turned up ScanWeaver ([arXiv 2606.00601](https://arxiv.org/abs/2606.00601), May 2026) — recurrences as a first-class abstraction, lowered to a single scan — which I cite proactively as the nearest adjacent work; it compiles everything to one backend rather than routing across four. What survived the search as unclaimed is only the boring part: **a dispatch table for TA indicators keyed on recurrence shape, including the willingness to route work *off* the accelerator when launch overhead wins.** The doctrine, not the primitives.

## The trade chain: throw away the linked list

The harder costume. A single-position backtest is a chain: each trade's entry is gated by the previous trade's exit. The first working GPU version modeled it honestly — as linked-list traversal, pointer-chasing from trade to trade. Sequential in a party hat.

The shipped version (`gpu_backtester.py`, March — the commit history calls it the culmination of seven attempts) replaces the traversal with a **Hillis–Steele prefix-max over exit barriers with per-pass re-selection**: each candidate entry's blocking barrier is a running maximum over what came before, which is associative, which means log-depth scan instead of length-of-chain steps. Prior art conceded here too: recursive doubling and pointer jumping are classic parallel-algorithms material, and Shen et al. ([arXiv 2205.13077](https://arxiv.org/abs/2205.13077)) parallelize the analogous activity-selection problem. What I couldn't find shipped anywhere is this instantiation: production backtesters either resolve chains sequentially (vectorbt, correctly) or keep per-timeline work sequential on the GPU on purpose. The record, pulled from the run database rather than my memory: on March 30, five discovery runs launched concurrently at 17:19 UTC — 100,200 strategy evaluations each — and all five completed by 17:43. **501,000 backtests in 24 minutes wall-clock, on a laptop GPU** (RTX 5000 Ada, 16 GB), roughly 345 backtests a second sustained. (I remembered it as "20 minutes"; the database says 24. The database wins — that's the house rule.)

## The part I'd actually defend

Fast and wrong is worthless in a money path, and everything above is the kind of cleverness that drifts. So the discipline from [the rest of this journal](2026-07-12-quarantine.md) applies with no exceptions:

- The GPU money path is held **byte-identical to the CPU reference** (1e-6), and the parity suite is what caught the classics — a fee applied twice, a cummax off by one — before they became folklore.
- The clever paths exist only as things an oracle checks. The naive implementations aren't deleted; they're retained as the referee, same pattern as [the prefix-invariance oracle](2026-07-16-the-one-that-survived.md).

Strip the provenance honestly and here's what's left: an agent with the parallel-algorithms literature in its weights, and an operator who wouldn't accept "it's sequential" as a final answer and wouldn't accept "it's fast now" without a bit-identical referee. Neither of us ships this alone.
