# Where four months of my GitHub graph went

*Published July 12, 2026. The events run December 2025 – July 2026; the discovery happened on the night of July 12. Dates, commit IDs, and quoted messages come from the git history. Cosmetic tweaks have landed since; the commits cited are the ones that built the thing.*

My local repos hold 6,230 commits since late December. On the night of July 12, my GitHub contribution graph credited me with 1,212 — and showed **zero** for February, March, and April, months I worked nearly every day. I assumed lost work. It was worse than that, and more interesting: three separate artifacts, stacked, each one hiding behind the next.

## Artifact one: the graph doesn't measure what you think

The monthly truth, from `git log` across my repos: December 31 commits, January 76, February 165, March 211, April 654, May 2,059, June 2,311, July 723 in twelve days. The May explosion isn't me working harder — it's the month my multi-agent setup matured and commit throughput tripled. Which means the graph's *intensity scale* recalibrated: next to a 2,000-commit month, a 200-commit month renders as faint noise. The graph stopped measuring my effort in February terms months ago. First lesson: a contribution graph is a headcount chart wearing an effort costume.

## Artifact two: I was three different people in January

My first repo's 89 commits carry three author identities: 56 under my real email, 28 under `dev@tradesite.local` — a made-up domain no GitHub account can ever verify — and the rest under an address that must never be attached to this account at all. GitHub only credits commits whose author email is verified on your account. The `.local` commits are unclaimable forever; they were written by a person who doesn't exist. If you're new: set your noreply email in git config on day one, before your first commit, and check it in every clone.

## Artifact three: the anonymization that ate the evidence

On June 29 I cleaned up those identity mistakes. The first attempt was gentle — a `.mailmap` commit (`8020bb414`), its message carefully noting "owner-directed, no history rewrite." The actual fix happened the same day, and it *was* a history rewrite: every author identity rewritten to my GitHub noreply address, every pre-rewrite SHA orphaned. I have the receipt because I made a backup first: in the bundle `tradesite-FULL-prerewrite-backup-2026-06-29.bundle`, the old main tip (`ad5aa9d72...`) is not an ancestor of the current main (`045712f0c`). A clean rewrite is invisible from inside the rewritten history — the commit that promised "no history rewrite" was orphaned by the rewrite it disclaimed.

Here's the part the docs could have told me: GitHub attributes contributions when commits are *pushed*, and a force-push of ~6,000 rewritten commits doesn't re-attribute them all — my calendar shows a single spike of 996, almost exactly the processing cap, credited to the push date. The rewrite traded four months of contribution history for the identity cleanup. Nobody noticed for two weeks because of the final gag: all my repos are private, and the "Private contributions" toggle was off, so the graph displayed nothing at all. The night I found this, flipping one checkbox took me from an empty graph to 1,212 — and no checkbox anywhere will recover February through April.

## What I'd tell you to do instead

1. **Day one:** set your GitHub noreply email in global git config. Audit `git log --format='%ae' | sort -u` in every repo before you have 6,000 commits.
2. **Before any history rewrite:** read how contribution attribution works. Commits are credited at push time under the email they carry *then*; a rewrite orphans the old credit and does not fully replay the new.
3. **Always bundle first.** My backup bundle is now the only proof of what the history used to say — including the only proof that the rewrite happened at all.
4. **Treat the graph as marketing, not truth.** Mine now permanently understates the record by a factor of five. The `git log` is the ledger; the lawn is decoration.

The punchline stands on its own: I rewrote history to protect my identity, and the rewrite quietly deleted the public evidence that I'd done the work. Privacy and provenance were in direct trade, and I spent one without noticing the price of the other. This post is me buying the provenance back the only way left — by publishing the receipts.
