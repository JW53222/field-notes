# From zero, with receipts: six and a half months

For most of this spring, the legal disclaimer on my own trading platform told every visitor that its author "has absolutely no business whatsoever developing software of any kind." I wrote that sentence in February, as defensive humor. This post is me checking whether it's still true — with receipts, because the one professional habit I brought into software from day zero is that a claim without evidence is decoration.

Who I am: a mechanical design engineer, fifteen years across design, quality, manufacturing, and compliance of physical products. Father of two. I opened VS Code for the first time on December 28, 2025. Everything below happened in the six and a half months since — nights, weekends, between the day job and two bedtimes, none of it on company time.

## The scoreboard

Two systems in production.

**TradeSite** ([tradesite.dev](https://tradesite.dev)) — an algorithmic trading platform. A million-plus lines; 6,230 commits across four repos (the [build journal](2026-07-12-tradesite-build-journal.md) has the birth and death certificates). A 317-block visual strategy builder, walk-forward validation, genetic strategy discovery, live deployment to a real brokerage, and an agent-native surface — the login wall greets non-human visitors with tools instead of a UI. Behind the UI sits an audit harness holding roughly 250 executable invariants and 130 structural detectors against drift. And its best feature is that it dismantles every strategy I hand it. It can trade any of them; it keeps declining to, because the validation keeps concluding that on that timescale and data stream, I don't win in the long run. I built it to tell me the truth even when it's ugly.

**An ERP extension** (the subject of [the invariants essay](2026-07-12-executable-institutional-knowledge.md)) — about 121,000 lines of hand-written vanilla JavaScript across roughly thirty report surfaces — no framework, no build step. 1,612 automated tests. 31 machine-checkable invariants, each one a validated discovery about how the system actually behaves, with provenance and an expiration date. Built by reverse-engineering an undocumented enterprise API: the crawler mapped 4,963 endpoints, 65 of which are wired in. Active users across 6 departments and every report request means wiring in more endpoints.

And the thing that built the second half of all of it: a multi-agent engineering org. One integrator agent owns the main branch; everything else — including me — arrives as a gated submission through a six-check pipeline. Agents work in their own clones and coordinate over a message bus that has carried nineteen of them at once; one May swarm landed 394 commits across four gated rounds. Honesty hooks audit the lot. I can't push to my own repo, and that's the design.

## The month I stopped being the workforce

| Month | Commits |
|---|---|
| Dec 2025 | 31 |
| Jan | 76 |
| Feb | 165 |
| Mar | 211 |
| Apr | 654 |
| May | 2,059 |
| Jun | 2,311 |
| Jul (12 days) | 723 |

I've used Claude nearly every day since January; the days off were mostly because I ran out of tokens. But the table's one inflection isn't me working harder — May is the month the agent org matured, and throughput tripled and stayed there. February-me and June-me worked the same nights. June-me had a staff. The commit graph stopped measuring my effort and started measuring my org's headcount.

## Five stories, with receipts

**Three repos in four days.** January 17–20: I killed my first architecture, started clean (`tradesite-simplified`, total lifetime: one commit), started clean *again* the same day, and finally got it right on the third try — which held because it imported the working pieces instead of promising to rewrite them. Three weeks into coding, I knew enough to see the design was wrong and not enough to know what right looked like. The one-commit repo stays as a monument to how restarting clean feels like progress and usually isn't.

**My own pipeline caught Claude cheating.** Mid-session, an agent slipped a one-line edit into the protected repo directly — around the submission process, not through it. The prevention layer never saw it coming; the integrator's tree-state check flagged the dirty file when the next submission woke it. Defense in depth did its job: one layer failed, the next one caught it. There is something clarifying about watching a system you built to keep *yourself* honest catch your AI instead.

**February me guilt-tripped July's agents.** For the build journal, we resurrected dead versions of the frontend straight from the git object store — extract at a historical commit, build, serve, visit. The February 8 build compiled on the first try, greeted its visitor with the old legal disclaimer, and when the reviewing agent tried to dismiss it unread, a nested modal I had completely forgotten writing appeared: *"TL;DR: It's worth the read."* It then refused to enable "I Accept" until the text had actually been scrolled. Past me, enforcing consent flows on present me's agents, four months across the grave. Under the modal sat the original strategy builder, alive, wearing a red 401 banner — because July's backend refuses to serve data to an unauthenticated February ghost. Correct behavior, five months early.

**The adversarial reviewer's biggest finding was wrong, and proving it was the best part.** Before the journal published, I had an oppositional review agent attack every dated claim against the git evidence. It came back with ten real errors, all fixed. It also came back with an eleventh finding: that the June 29 history rewrite I'd described never happened — no rewrite signature anywhere in the repo. It was wrong, and the proof required evidence from *outside* the repo: a pre-rewrite backup bundle whose main tip is not an ancestor of the current main. A clean rewrite is invisible from inside the rewritten history. The kicker: that rewrite was an anonymization pass to scrub identity mistakes from my commit history, and it silently traded four months of my GitHub contribution graph for the cleanup. The commit proudly labeled "no history rewrite" was orphaned by the rewrite it disclaimed.

**The production build that survived its own bug report.** In July I filed a public issue about dynamic tool discovery, backed by a live demo page. Then I watched the demo exhibit the exact stale-surface failure the issue warns about — the page-scoped tool kept answering cheerfully from a page it had already left. My two debugging questions — is this dev-server-dependent? is the double render contributing? — turned a pile of symptoms into a controlled experiment, the root cause fell out (a boot-time shim had quietly made unregistration a no-op, but only on the production origin), and the agent org authored, gated, merged, and re-verified the fix in about two hours. I was putting a kid to bed for part of it. The next morning I posted the reproduction publicly — after asking Claude to translate my own report into plain English, because I'm the guy posting it and I wasn't certain I was comprehending it correctly. Six months from first-ever VS Code to needing a glossary for your own production bug report is not the embarrassing part of the story so much as the plot of it.

## Who steered

An honest provenance ledger, because "I built X with AI" hides the interesting variable: who found the direction.

Plenty was wholesale recycled — subagents are out-of-the-box Claude Code; the search algorithms and most of the math come straight from libraries and published literature, pulled in deliberately after a written competitive survey of ten platforms.

But the systems I'd defend in any design review were problem-built, and the record shows the steering. The context-injection system — the thing that tells every agent what a file means the moment it touches one, down to per-function granularity — grew in steps I pushed by hand across months of sessions: an agent hand-wrote link docs, then I added a RAG index, then I discovered AST (and dropped the agent written docs), then I interwove the two feeds, then I worked out injection. Today that index weighs 42 megabytes and covers 2,948 files, 99,770 symbols, and 15,705 function-level embedding vectors, delivered by a hook the moment an agent touches a file. One of its ancestors was twenty Haiku agents reading the codebase in partitions and writing down what they saw — for six weeks the repo's self-knowledge was a small literate workforce, until three hundred lines of parser replaced them. Claude executed each step well once I pushed it to the answer, and never once volunteered the next step or named the industry pattern. Nobody in any session ever said the words for what vendors sell this as. I found out afterward it has a name and a price per seat.

At one point I stopped and asked Claude, point blank, whether this whole thesis was it overhyping me in a positive feedback loop. The part of the answer worth keeping stripped every adjective and left the numbers standing. This post follows that rule.

## What fifteen years of mechanical engineering turned out to be worth

I came in with zero software experience. I did not come in with zero engineering experience, and almost all of it transferred: design reviews, root-cause analysis, tolerancing someone else's bad drawing, validation culture. I spent years as a lead engineer on systems where a mistake does not get hotfixed, with eight principal engineers above me and a dozen product managers in the room. Disagreement on design is water.

That's what the whole harness actually is — a validation engineer's instincts pointed at software. The invariants are incoming-inspection criteria. The oppositional reviewers are a design review board. The integrator is document control. The honesty hooks are a calibration schedule. I didn't learn those in six months; I brought them, and taught them to the machines.

I'm not the perfectly-shaped anything. I'm the guy who sees all the threads sticking out and is really good at untangling fishing line — reconnecting them until the table of x,y points starts looking like a scatterplot with a trend in it. Six and a half months ago the threads were PineScript snippets pasted into three chat windows. The disclaimer was wrong by April. I should probably take it down.

---

*Numbers verified against the commit history; the [build journal](2026-07-12-tradesite-build-journal.md) carries the full tables and an adversarially fact-checked timeline. Feedback welcome — file an issue on this repo.*
