# The duplicate detector lives in the write loop

*Published July 12, 2026. The events run April 21–26, 2026, with the origin story reaching back to March. Dates, commit IDs, and quoted messages come from the git history. Tweaks have landed since; the commits cited are the ones that built the thing.*

Everyone puts embeddings in front of the model: semantic search as retrieval, so the agent can find code before it writes. I ended up also pointing them *backwards* — at the model's own output, inside the merge gate — because of a problem every multi-agent codebase eventually meets: **agents reinvent your helpers.** Not copy them — reinvent them, slightly differently, under a new name, in a plausible location. Each duplicate is invisible to grep, splits future callers into camps, and doubles the surface every later agent has to read. And reading is the expensive operation: the whole reason I had embeddings at all is that it cost too much to have Claude re-read the codebase to find things out.

## Built in a day, verified for a week

The harness landed on April 21 in a single day's work (`b213bc528`): executable invariants, structural detectors, **semantic detectors** — embedding-based checks that compare what an agent just wrote against what already exists — and drift tracking, together as one audit pipeline. The next day's commit is the origin story writing itself: "Rebuild audit-hardening work lost after session…" — the harness's first production act was recovering its own lost construction.

Then the tuning, which is where the honesty lives. April 24: the embedding cache moved to float16 (“8x smaller, ~20x faster”), because a detector that's too slow to run on every merge is a detector that doesn't run. April 26: five triage waves in one day, each one adversarially reviewed, and the commit messages report their own null results — "triage wave-11 — 3-lane detector improvements, **0 real bugs**" (`7c271e422`), "wave-13 — incorporate deferred oppositional findings" (`77d64bc2f`). A detector suite whose changelog admits *this wave found nothing real* is a detector suite you can calibrate. One whose every wave claims victory is a slot machine.

## What it actually does at merge time

When work arrives at the gate, the semantic detectors embed the new symbols and compare them against the corpus of everything already in the tree. High-similarity hits aren't auto-rejected — similarity is a lead, not a verdict — they're surfaced to the review step with the existing twin named, so the adjudication is "justify the second copy or call the first one." The effect compounds: every duplicate that doesn't land is context the next hundred agents don't have to read, which in an agent codebase is the scarcest resource there is.

The same vectors turned out to be the seed of something bigger. Deduplication was the first consumer; then the corpus got merged with the dependency map and injected into agents' context at file-touch time, and by the time I learned the industry sells that combination as "hybrid retrieval," I'd been running it for weeks (the [longer story](2026-07-12-from-zero-with-receipts.md) has that lineage, with the numbers).

## The claim, corrected the same day

The first version of this post ended with "nothing I've found audits what the model writes against what it already wrote." Then I sent an adversarial research agent to refute me, and it did: [Pharaoh](https://pharaoh.so/blog/prevent-duplicate-functions-ai-coding/) published PR-time signature-twin auditing for AI coding workflows on February 26, 2026, and [Octopus Review](https://octopus-review.ai/blog/your-ai-writes-the-same-code-5-times) published embedding-based semantic dup detection over the whole repo — Qdrant vectors, write-side, at the PR gate — on March 30, 2026. Both make the same read-side-versus-write-side argument this post makes. Both predate my April 21 build by weeks. I'd seen neither.

So the honest claim is smaller and, to me, better: this is a product category that was being born in the spring of 2026, and I converged on it independently, in five days, seventeen weeks into software, from nothing but token economics and the lived annoyance of agents reinventing my helpers — as an internal stage of my own multi-agent merge gate rather than a standalone PR bot. Parallel invention weeks apart isn't a first-place trophy. It's evidence the constraints were speaking clearly and I was listening. I'd rather own that sentence than the overclaim it replaced.
