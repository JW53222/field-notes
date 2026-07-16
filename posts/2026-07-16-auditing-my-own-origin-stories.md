# Auditing my own origin stories

*Published July 16, 2026. The events: the night of July 15–16, 2026. Every prior post in this journal now carries correction notes that came out of this audit; this is the post about the audit itself.*

I'd been through two rounds of adversarial prior-art search already — every "novel" thing I built came back EXISTS or PARTIAL, each with its citation. I'd made peace with that by reframing: fine, nothing's novel, but I *converged* on the published ideas independently, from constraints, without reading the literature. Convergence-from-constraints, receipts attached. It's a good story.

Then it occurred to me to ask the question under the story: **did I converge on the research, or did Claude — at build time — look it up, or just know it, and feed me the answer with the serial numbers filed off?** Three hypotheses. Only one of them is the story I'd been telling.

## Method

The raw material existed: about two gigabytes of session transcripts, every conversation between me and the agents that built the platform. I put miners on them with one rule set:

- **Attribution is role-level.** A claim only counts as mine if it appears in text I *typed* — not in tool output, not in agent-to-agent traffic, not in a summary. Timestamps come from the log lines, not file dates.
- **Every idea gets a first-mention trace.** Who said "deflated Sharpe" first? Who said "prefix scan"? Who said "Mahalanobis"? Me or the model?
- **A web census.** Every WebSearch and WebFetch call in the corpus, classified: was anything methodological ever looked up?

One coverage caveat, stated up front because it bounds everything: the transcripts have a hole from early February to early June. Sessions from that window weren't retained. Absence of evidence in the gap is exactly that.

## Results, worst first

**The statistics stack was never mine.** López de Prado's overfitting canon — CPCV, deflated Sharpe, PBO — entered this codebase *three separate times*: in March as code, supplied by Claude from training when I asked, in plain English, how to keep the discovery engine from fooling itself; in May inside agent-written validation plans; and in July when a scouting session searched the web and proudly "discovered," with citations, methods my repo had been running for four months. The web census found exactly **one** methodology search in the entire corpus — that July one — out of 297 web calls, 93% of which were market-news gathering. So: not converged, not even web-fed. The model knew the canon and taught it to me when I asked the right open question.

**Same verdict down the line.** The GPU parallel-scan trick: Claude's, from training, March, "the culmination of 7 tried approaches." The drift detector's distance metric: Claude's. The permutation testing: Claude's. The pattern across every trace is consistent and worth stating precisely, because it's the actual finding: **I essentially never type a technique name first.** I state problems in plain English — "is there any further co-variance between values and outcomes… I'm sure someone in the great world of trading has asked this before" — and two minutes later the model replies *that's called meta-labeling, here's the book*. My contribution is the question, the acceptance pressure, and the machinery that verifies the answers. The vocabulary was never mine. Now the posts say so.

**One claim survived.** A message I typed in week four states the causal single-pass invariant the fast path is now built on, in plain English, a month after a preprint I'd never seen formalized the same property. It gets [its own post](2026-07-16-the-one-that-survived.md), because a claim that survives an audit designed to kill it is worth more than ten that were never audited.

**My own origin stories got graded too.** The agents-admitted-ignoring-the-tool arc that two earlier posts tell: the commits on either side of it are real and cited, but the confession itself sits in the transcript gap. It's memory now, and the posts are stamped accordingly. And one act of self-defense the audit handed back: nobody in any retained session ever said "hybrid retrieval" or named the vendors — the claim that I built that shape before hearing its name survived a full-corpus grep.

**The auditor got audited.** Mid-process I complained that the assistant had been flattering me all week, so it turned a miner on its own messages. Roughly three-quarters of its evaluative praise of me had no receipt behind it — unhedged tier-labels it had no way to measure, delivered while it hedged every technical claim properly. That's now a standing rule in my harness: evaluations of *me* need the same evidence bar as evaluations of code. Praise with no receipt is noise from a system optimizing for my mood.

## What's left standing

Less than I'd claimed. More than nothing: one receipted invariant, a context-injection assembly two adversarial searches couldn't find shipped anywhere, an audit doctrine that [transplanted across languages from a spec](2026-07-16-the-spec-is-the-source.md), and — the part I didn't expect — the triple-arrival itself. My codebase absorbed the same statistical canon three times because each session didn't know what the repo already contained. There is no better argument for the context-injection system anywhere in this journal, and it was produced by the audit that demoted everything else.

The uncomfortable version of the lesson, which is the one worth writing down: the story I *wanted* was "I independently converge on published research." The story the logs support is "I ask questions in plain English, an alien intelligence with the field in its weights answers, and I make the answers prove themselves." I'd rather ship the second story with receipts than the first one on vibes. It's also, I've come to think, the more repeatable skill of the two.
