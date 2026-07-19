# Field Notes

Notes from an accidental software engineer.

I'm a mechanical design engineer at a make-to-order manufacturer. I opened VS Code for the first time in December 2025; since then I've been building tools that make hostile, undocumented systems tell the truth — a browser extension six departments use daily against our ERP, and an agent-native trading platform. These are the write-ups of whatever survived contact with production.

## Posts

- **2026-07-16 — [The scoreboard: every claim in this journal, tiered by what it survived](posts/2026-07-16-the-scoreboard.md)**
  The grading system every claim here passes through before publication, and where each claim sits.
- **2026-07-16 — [Parallelizing the sequential: recurrence shapes, and a trade chain on a GPU](posts/2026-07-16-parallelizing-sequential-recurrences.md)**
  Route every stateful indicator by the shape of its recurrence — including off the GPU when launches dominate — and replace trade-chain traversal with a log-depth prefix scan. 501,000 backtests in 24 minutes on a laptop GPU, held bit-identical to the slow reference.
- **2026-07-16 — [Borrowing your own login: scripting an SSO-gated enterprise API with zero stored credentials](posts/2026-07-16-borrowing-your-own-login.md)**
  Attach to your own logged-in browser over CDP and ride the session — with permission, at my own employer, for read-only pulls. Zero credentials at rest, full attributability, and the sharp edges stated.
- **2026-07-16 — [Auditing my own origin stories](posts/2026-07-16-auditing-my-own-origin-stories.md)**
  I mined two gigabytes of my own session transcripts to trace where every load-bearing idea actually came from. What got demoted, what survived, and what the logs say my job actually is.
- **2026-07-16 — [The one that survived: an invariant I typed before I knew its name](posts/2026-07-16-the-one-that-survived.md)**
  Four weeks into ever writing software, I typed the causal single-pass property in plain English — a month after a preprint I'd never seen formalized it. The receipt, and the oracle that now enforces it.
- **2026-07-16 — [The spec is the source: my audit harness crossed languages without its code](posts/2026-07-16-the-spec-is-the-source.md)**
  A porting guide addressed "to a Claude agent in a different codebase" regenerated my Python audit harness in vanilla JS in a day. n=1, with dated receipts.
- **2026-07-12 — [From zero, with receipts: six and a half months](posts/2026-07-12-from-zero-with-receipts.md)**
  Start here. What a mechanical engineer and father of two actually shipped in 6.5 months from a first-ever VS Code install — the scoreboard, five stories with receipts, and a ledger of who steered.
- **2026-07-12 — [Where four months of my GitHub graph went](posts/2026-07-12-where-four-months-went.md)**
  I rewrote my git history for privacy and it quietly deleted the public evidence of the work. Three stacked artifacts, a backup bundle as the only proof, and what to do instead.
- **2026-07-12 — [Quarantine: deleting code without trusting your own analysis](posts/2026-07-12-quarantine.md)**
  Dead code gets instrumented, not deleted — deletion is earned by 30 days of runtime silence. Out-of-band edits get caught and held, never lost. Prevention is a hypothesis; detection is a fact.
- **2026-07-12 — [The duplicate detector lives in the write loop](posts/2026-07-12-dup-detection-write-loop.md)**
  Embedding dedup inside the merge gate, built in five days — then an adversarial prior-art search found the product category being born the same season. Independent convergence, corrected same-day.
- **2026-07-12 — [Survey the workforce: user research where the users are Claudes](posts/2026-07-12-survey-the-workforce.md)**
  I built a lookup tool and my agents admitted to ignoring it — so I switched to injection, they dodged the trigger, and the harness answered. Evasion is the sincerest survey response.
- **2026-07-12 — [Two dates for WebMCP: the gateway and the front door](posts/2026-07-12-webmcp-two-dates.md)**
  I shipped a tool surface for visiting agents in February, killed my own chatbot in April, and pulled the tools in front of the login wall in July — early adopter, not inventor, with the lineage cited.
- **2026-07-12 — [Executable institutional knowledge: regression-testing a system you can't read](posts/2026-07-12-executable-institutional-knowledge.md)**
  How 31 provenance-tracked, machine-checkable invariants turned forensic ERP discoveries into CI rules — and why this matters more when an AI assistant writes most of your code.
- **2026-07-12 — [Build in public, retroactively: seven months of TradeSite, reconstructed from the commit history](posts/2026-07-12-tradesite-build-journal.md)**
  A devlog reverse-engineered from 6,230 commits: birth and death certificates for every system, time-machine builds of dead pages, and a fact-check of the whole album against the git evidence.

Feedback: open an issue.
