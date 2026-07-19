# The scoreboard: every claim in this journal, tiered by what it survived

*Published July 16, 2026. The grading system every claim now passes through before publication, and where each claim sits. It exists because two prior-art search waves and [a provenance audit of my own transcripts](2026-07-16-auditing-my-own-origin-stories.md) kept demoting things I'd have otherwise overclaimed, so I wrote the rules down and put everything through them.*

## The rules

1. **The prior-art bar is the union** — shipped products, open source, blogs, *and* research preprints. The harshest available bar: an idea "exists" if anyone findable ever wrote it down. Note the reverse implication: "anticipated by a paper" is **not** "you could have installed it." Where that distinction matters, I state it.
2. **Provenance is a separate axis from novelty.** A thing can be unanticipated *and* not mine (my agent derived it), or fully anticipated *and* genuinely mine (I stated it before learning its name). Both axes get graded; conflating them is how people lie to themselves in both directions.
3. **Attribution is role-level.** Personal origin requires text I *typed*, timestamped, in the session logs. Recall — anyone's — doesn't count; the logs and the git record do.
4. **Demotions get published** in the same channel as the claims, dated.

## First-shipped instantiation
*The idea was discussed or adjacent in public — but no shipped implementation was found. The claim is the instantiation and its date, never the concept.*

- **Inject-on-touch context push + post-write duplicate check, as one loop** (May 27 / June 2–3). Nearest shipped neighbors, verified by reading them: Aider's repo-map (push, computed — but whole-repo at prompt time, no per-file hook trigger, no embeddings) and Cursor's glob rules (push-at-touch — but hand-authored static prose, not computed bundles). [Post.](2026-07-12-survey-the-workforce.md)
- **Dedup bracketing the edit inside the merge pipeline** — gate advisory + whole-tree sweep, twins handed to adjudication. Concept conceded to Greptile; the locus survived two review passes. [Post.](2026-07-12-dup-detection-write-loop.md)
- **The anonymous agent front door** — logged-out routes where context tools answer honestly and gated actions return structured *account-required*. [Post.](2026-07-12-webmcp-two-dates.md)
- **Audit harness regenerated from spec across languages** (April 22→23, Python→vanilla JS, n=1 declared). [Post.](2026-07-16-the-spec-is-the-source.md)
- **Four-family recurrence dispatch for TA indicators** — every primitive is literature; the routing table survived search (ScanWeaver cited as nearest adjacent). [Post.](2026-07-16-parallelizing-sequential-recurrences.md)

## Early adopter
*Concept public first; shipped here before mainstream availability.*

- **WebMCP gateway in production, February 13** — Chrome's origin trial announced May 19. Lineage fully cited in [the post](2026-07-12-webmcp-two-dates.md); "early adopter, not inventor" is that post's own phrasing.
- **Tombstone-style quarantine with AI-pipeline nomination** — the pattern is a decade old (conceded: scheb/tombstone, Uber Piranha); the nomination source and versioned ledger are the delta. [Post.](2026-07-12-quarantine.md)

## Fast-follow, convergent with a category being born
- **Write-loop embedding dedup** (April 21) — Greptile shipped earlier; Octopus Review's first release beat mine by four weeks. Claimed as independent convergence *on the category*, corrected same-day when the search said so. [Post.](2026-07-12-dup-detection-write-loop.md)

## Literature-applied, deliberately
*No convergence claim of any kind. The skill claimed is selection and honest implementation.*

- **López de Prado overfitting stack** (CPCV/DSR/PBO) — supplied by Claude from training in March when I asked how to stop fooling myself; entered the repo three separate times. The code labels its own approximations as not-the-real-statistic. [The audit post](2026-07-16-auditing-my-own-origin-stories.md) tells the triple-arrival.
- **LLM plateau-seeder** — design doc cites its own lineage (LMX, Eureka) at birth; EvoGPT is near-identical prior art. Per the git record: the *problem* is receipted March 18 (a "population diversity collapse" troubleshooting section in the discovery docs, plus rules-based hybrid seeding landing the same day), and the LLM answer is first receipted April 26 — design doc and NSGA wire-in three commits apart. An earlier conversational origin can't be established either way (transcripts from that window weren't retained), so none is claimed. The claim: "grounded in the literature at design time, gated by the production validator."
- **Permutation testing** — Westfall–Young is 1993 textbook statistics. Cited, not claimed.

## Claude-from-training, steered
*The algorithm was never mine; the problem statement, the routing doctrine, and the oracle were.*

- **GPU prefix-max trade-chain kernel** (March, "culmination of 7 tried approaches" — the approaches were the agent's; the refusal to accept "it's sequential" was mine). [Post.](2026-07-16-parallelizing-sequential-recurrences.md)
- **Mahalanobis trigger-drift detector** (April) — standard multivariate distance on trigger-setup features; the anti-false-green framing around it is the house discipline.

## Receipted personal origin
*Typed by me, timestamped, before I knew the name.*

- **The causal single-pass invariant** — January 25, week four, plain English; formalized by the model over months; enforced by a whitelist and a bit-identical oracle; anticipated (unread) by a December preprint. The one claim that survived the audit built to kill claims. [Post.](2026-07-16-the-one-that-survived.md)

## How-to, no claim
- **CDP auth-borrowing fetch** — Playwright/Puppeteer territory, conceded entirely; published as technique. [Post.](2026-07-16-borrowing-your-own-login.md)

---

One reading of this table is deflating: the top tier is narrow engineering assemblies and everything glamorous sits in lower tiers. The other reading is that every row survived a process that was genuinely trying to move it down — and the rows that moved, moved in public. I know which reading I trust.
