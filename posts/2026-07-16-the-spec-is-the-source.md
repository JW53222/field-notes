# The spec is the source: my audit harness crossed languages without its code

*Published July 16, 2026. The events: April 12–23, 2026. Dated receipts are embedded timestamps in the two repos' own documents; both repos are mine, which is exactly what makes the experiment clean.*

There's a future-state idea circulating in the AI-coding discourse: the specification becomes the source artifact, and the code becomes a build product you can regenerate — different language, different environment, same system. Tooling proposals exist (GitHub's spec-kit is the well-known one). Mostly it's talked about as where things are heading.

In April I did it by accident, and only realized this week — during [a provenance audit](2026-07-16-auditing-my-own-origin-stories.md) that was busy demoting my other claims — that the receipts for it were sitting in two repos the whole time.

## The setup

My trading platform has an audit harness: a few hundred invariants, coverage accounting as the product ("considered ⊇ parsed ⊇ checked" — the question isn't just what failed, it's what was never examined), detectors that audit themselves, zero-evaluations-is-yellow. It's Python, grown organically over months, entangled with its codebase the way real systems are.

My day job has an entirely unrelated codebase: a vanilla-JavaScript browser extension against an enterprise ERP. Different language, different domain, different machine, nothing shared but me.

I wanted the discipline over there too. The obvious move is to copy code. The code was uncopyable — wrong language, wrong shape, wrong everything.

## What actually crossed

A document. I had an agent write `AUDIT_SYSTEM_PORTING_GUIDE.md`, and its audience line is the whole idea: **"Audience: a Claude agent in a different codebase."** Not a human maintainer. Not a build system. A future agent, somewhere else, holding a different toolchain, expected to *regenerate* the system from the doctrine: the six-layer model, the coverage-accounting contract, the envelope schema, the orthogonality test for new detectors, what counts as a finding, what counts as silence.

The embedded timestamps tell the rest. The guide carries the source system's own run artifacts from **April 22**. The ERP-side port's first calibration triage is dated **April 23** — one day later. The regenerated harness came up in JavaScript, ran its first five invariants, and the doctrine held: same accounting, same yellow-not-green rules, same shape, zero lines of shared code. It's been catching real bugs in that codebase since (that system has [its own post](2026-07-12-executable-institutional-knowledge.md), which now carries this lineage note).

## The claim, at its correct size

Both repos are mine, so this is not a story about cross-team knowledge transfer — no politics, no onboarding, no resistance. That's a limitation and also what makes it a clean experiment: the only thing that crossed the gap was the document, so the document is the thing that worked.

And it's n=1. One harness, one transfer, one day. I'm not claiming "regenerable in general"; I'm claiming **transferable from spec, demonstrated once, across languages and domains, with embedded dates** — in April, while the pattern was still mostly a slide in other people's decks. Early instantiation, not invention. Flying cars have been conceptualized for as long as there have been cars; the interesting date is always the first time one actually leaves the ground with receipts, however short the hop.

The part I'd generalize, carefully: the harness turned out to be *made of doctrine, not code*. Everything that survived the crossing was a rule about evidence — what must be accounted for, what may never pass silently. Everything that stayed behind was implementation. If your system's value survives a one-day rewrite into another language by an agent that's never seen the original, you've learned what your system actually is. Mine was a contract wearing a Python costume, and the contract is portable.
