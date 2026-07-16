# The one that survived: an invariant I typed before I knew its name

*Published July 16, 2026. The events: January 25, 2026 (the message), May–June 2026 (the formalization), July 15 (the audit that found the receipt). The quote below is copied verbatim from a session transcript, typos and all.*

I spent July 15 running a provenance audit against my own resume claims — the [full story of that night has its own post](2026-07-16-auditing-my-own-origin-stories.md), and the short version is that most of my "I converged on this independently" claims died under it. This post is about the one that didn't.

## The message

On January 25, 2026 — four weeks after I opened VS Code for the first time — I was unhappy with how the platform detected liquidity events. The naive approach re-scanned history on every new candle. I typed this into a session:

> "full history in one scan's not good. it needs to detect if a candle has set a new event ... process bars sequentially once ... maybe just keep an index of events ... you figure out an efficient way to do it."

That's the whole artifact. No technique names, because I didn't know any. What it contains, in plain English: process the series **once, forward, sequentially**; maintain an **index of events** instead of re-deriving them; never re-scan. Reject the O(n²) shape, demand the O(n) one, delegate the implementation.

Claude formalized it in the following turns as event-lifecycle tracking and wrote the first tracker that session. Over the next five months the same property kept getting more formal clothing, all supplied by the model, none by me: "causal," "no-lookahead," and eventually **prefix-invariance** — the requirement, now in a docstring and a parity test, that the last element of the j-th prefix run equals element j of a single full-series pass. Blocks that satisfy it are provably free of look-ahead; blocks that don't (structures back-stamped by future bars) provably aren't.

## What the repo does with it

This is the part I'd defend in a design review, because it's where the invariant stopped being a sentence and became enforcement:

- Indicators are **whitelisted onto the fast path only if they pass the prefix-invariance check.**
- The O(n²) per-prefix loop wasn't deleted when the O(n) accelerator shipped. It was **retained as a bit-identical oracle** — the slow correct thing that fails the build if the fast thing ever drifts. The naive version's only remaining job is to keep the clever version honest.

## The prior art, conceded first

In December 2025 — about a month *before* my message — Saggese & Smith posted DataFlow ([arXiv 2512.23977](https://arxiv.org/abs/2512.23977)), which treats the same territory formally: point-in-time idempotency and tilability as the properties that make streaming and batch evaluation agree. I had never seen it. The provenance audit checked: the paper is named nowhere in any session, and the sessions in question made zero web calls. (Verified against the paper's abstract, not a theorem-by-theorem mapping — in keeping with this journal's general paranoia about its own citations.)

So the claim, stated at exactly its size: I did not invent this property, and a preprint beat my message by a month. What I did was **state it in my fourth week of ever writing software, from the problem alone, and then build a system where it's enforced by an oracle rather than remembered by a person.** The formal vocabulary was Claude's, from training. The property was mine, from the tape.

## Why this one gets a post

Because of the direction of the audit that produced it. On July 15 I set out to *demote* my claims — miners over two gigabytes of my session transcripts, attributing every load-bearing idea to whoever typed it first. The de Prado stack, the GPU scan tricks, the seeder: all demoted to "Claude supplied it, I steered." This receipt went the other way. A claim that survives the process built to kill it is the only kind I'm interested in publishing anymore.
