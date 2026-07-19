# Build in public, retroactively: seven months of TradeSite, reconstructed from the commit history

I never kept a devlog. I didn't know I should, turns out I didn't need to — 6,230 commits across four repos kept one for me. This post reconstructs it: what I built when, what replaced what, and death certificates for everything that didn't make it. Every date comes from a commit. An adversarial review agent then attacked every claim against the git evidence before I published; it caught ten errors, and I caught one of its own.

Cast: me — a mechanical design engineer who opened VS Code for the first time on December 28, 2025 — and a rotating team of Claude models that eventually became an organization of them.

## Chapter 0: PineScript (Thanksgiving 2025)

This started with me pasting PineScript between TradingView and the chat windows of ChatGPT, Gemini, and Claude.ai after Thanksgiving 2025. No repo, no editor. The fossils survive on my GitHub: a repo literally named `Junk` (Dec 19), and `pinescriptv6-private` (Dec 29) — created one day after I installed VS Code, because copy-paste had become the bottleneck.

The lesson that drove everything after: the chat window was never the product. The loop was — describe, generate, test, correct. Everything since has been work to make that loop faster, safer, and more honest.

## The four TradeSites: a family tree

| Repo | Born | Died | Commits | Cause of death |
|---|---|---|---|---|
| `TradeSite` | Dec 29 | Jan 15 (last commit; superseded Jan 17) | 89 | The architecture couldn't carry the ambitions. |
| `tradesite-simplified` | Jan 17 | Jan 17 | **1** | Stillborn. One commit: "Clean TradeSite architecture with Massive API + DuckDB." The clean restart that immediately proved too clean. |
| `tradesite-reborn` | Jan 17 | Jan 20 | 5 | Second restart the same day. Lived three days — long enough to prove the aggregation/scheduler design. |
| `TradeSite-unified` | Jan 20 | alive | 6,135 | — |

January 17–20: three repos in four days. Three weeks in, I knew enough to see the first architecture was wrong and not enough to know what right looked like. I answered "start over, keep what works" twice in one day. The third attempt held because it imported the working pieces instead of promising to rewrite them.

## Cadence

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

One inflection dominates the table: the month the multi-agent organization matured (May), commit throughput tripled and stayed there. In February I wrote features. By June I ran a system that wrote features.

## What Claude brought vs. what the problems forced

Before the track-by-track history, an honest accounting, because "I built X" means three different things in this repo and I want to use the right one each time.

**Wholesale recycled.** Subagents are out-of-the-box Claude Code. Search algorithms, most of the math — RSI to DSR — come straight from libraries and literature. When the overfitting guard suite went in this July, it went in as López de Prado's published methods, deliberately, after a written competitive survey (`docs/Planning/2026-07-06-competitive-landscape/RESEARCH.md`) that tore down ten platforms and verified every candidate idea against my codebase before anything got built. Its funniest finding: the first feature it investigated already existed in my repo, silently disabled. Most of its verdicts came back "PARTIAL, not NOVEL." *(And a July 15 provenance audit of my own transcripts made the joke recursive: the de Prado canon itself had already entered the codebase on March 15 — `1d2e11a82`, the first overfitting detector — and again in May, inside agent-written validation plans. The July survey went to the open web and "discovered," with citations, methods my repo had been running for four months. Three arrivals of the same canon, because sessions don't know what the codebase already holds. I keep telling people that's why the context-injection system exists.)*

**Problem-built, and I can show the scar tissue.** The context-injection system did not descend from any product. It grew in steps I steered by hand across months of sessions: first an agent hand-wrote link-and-summary docs; then I added RAG indexing over the code; then I interwove the two; then I worked out how to inject the merged map into an agent's context at the moment it touches a file. Claude executed each step well once I pushed to it — and never once suggested the next step or named the industry pattern. I found out afterward that vendors sell this shape as "hybrid retrieval." Nobody in any session ever said the words "Sourcegraph Cody" to me.

Same story for the agent org. The message bus: when I later went looking for something better to replace it, the search came back empty — nothing beat it, so I hardened it instead. The gated-merge structure: I had been running oppositional reviewers as pre-commit gates well before there was a proper CI chain, and the integrator design came from asking what a real software team should look like — forced by agents overwriting each other's work, which is also why every agent works in its own local clone. The agent-sizing rules started as the same paragraph I kept retyping into prompts; on May 12 I finally wrote it down as a skill. Its opening line is pure lived experience: the repo's default had been "Opus + worktree for everything," and that "burns 3-4× the tokens needed."

The pattern across every big custom build: market research first, then build the delta. I would have happily pulled in existing solutions — when we looked, especially in the freeware space, there was rarely anything to pull.

## Track 1: The data layer — three generations

| Date | Event |
|---|---|
| Dec 29–31 | yfinance + Polygon.io flat files |
| Jan 15 | I migrated Polygon.io → Massive API |
| Jan 17 | DuckDB became the store (per `simplified`'s one commit) |
| Feb 23 | Encrypted DuckDB threw double-attach errors "across all endpoints" — the first sign an embedded database and a concurrent web app disagree |
| **Mar 15** | **Replaced DuckDB with PostgreSQL/TimescaleDB** |
| Mar 22 | Deleted 18 deprecated DuckDB-era scripts — the funeral |
| Mar 24 | Automated backups + WAL archival, landing within 48 hours of losing a session's work (see the failure ledger). I learned that paranoia the hard way. |

## Track 2: The engine

| Date | Event |
|---|---|
| Jan 2 | First "comprehensive backtest system" (old repo) |
| Jan 25 | VectorBT backtesting, IBKR integration, smart-money analysis land in unified |
| Feb 13 | Multi-timeframe backtester; multi-position backtests |
| Feb 24 | Live execution exit-strategy support |
| Mar 24 | The IBKR socket client (ib_async) went in as a drop-in beside the REST client, ran in parallel-validation mode, and replaced it the same day ("Phase 4 — remove REST client, socket-only") |
| Apr 8 | Solver extraction: signal generation and trade resolution became standalone packages, with the shared eval package following the next day |
| Apr 21+ | The parity project: a reference-predicate registry as an independent oracle against the engine, agreement tracked commit-by-commit (18.7% → 34.1% → …) |
| May 17 | Live-risk gates hardened after an oppositional review |
| Jun 17 | Portfolio backtesting |
| Jul 6 | Overfitting guard suite (DSR/PBO/CPCV) — built as the top item out of the competitive survey, wired into genetic discovery, exposed as an endpoint |

## Track 3: The frontend — and why it's called SuperComboNew

The confession first. `SuperCombo` — the strategy-builder page — was born in the old repo on January 6 ("Phase 5: Super-Combo Block Library Implementation"). On February 9 at 07:41, a commit titled "Major platform expansion" added `SuperComboNew.tsx`, `ControlBarNew.tsx`, and `StrategySectionNew.tsx` beside it. Sixteen hours later, at 23:23, "Overnight platform overhaul" deleted the original. The suffix was accurate for sixteen hours. I swept the migration shims out on March 22, by which point "New" described nothing — but the name was load-bearing in tests, routes, and muscle memory. It has now outlived its predecessor by five months. Every codebase has one of these; mine is the flagship page.

State management moved twice in nine days during migration season: props → `SuperComboContext` (Mar 15) → Zustand store (Mar 24).

Page births by month (every one still alive):

| Month | Pages born |
|---|---|
| Jan | Chart, DataAdmin, IBKR Account/Login, Tools, Futures |
| Feb | Glossary, Login/Register, **SuperComboNew**, MarketScanner, News, StrategyDashboard, UserManagement, Discovery, Account, Community (×2), Forum (×3), Leaderboard, Monitoring, PublicProfile |
| Mar | Pricing, Billing Success/Cancelled, Terms, TrainingDashboard, Welcome, WorkerSetup, ForgotPassword, LessonPlayer |
| Apr | Billing, BillingAddons, IndicatorProfiler |
| May | NotFound (day 135: the project admits URLs can be wrong), ProMode |
| Jun | PITScreener, PortfolioBacktest |
| Jul | DemoLanding, DynamicToolTest — pages built for visitors who aren't human |

February 27 alone birthed nine pages — community, forum, leaderboard, profiles. That's the day TradeSite decided it was a platform, not a tool.

## Track 4: The agent surface — the pivot

| Date | Event |
|---|---|
| Feb 13 | I shipped an in-site assistant chat (a Claude chat embedded in the app) and the **WebMCP gateway**, same day |
| Apr 16 | **The pivot.** Two commits, hours apart: "advisory-only AI-agent surface across TradeSite," then "remove in-site Claude chat + all its plumbing" |
| Apr 16–17 | Rewrote the tool catalog in descriptive voice; closed two waves of audit findings against the agent surface |
| Jul 11 | Registered for Chrome's origin trial; shipped the anonymous front door with a dynamic-tool demo; root-caused and fixed the origin-trial unregister bug through the merge gate — one day |
| Jul 12 | Posted a public console repro of the dynamic tool list |

April 16 is the clearest strategic decision in the whole history: kill the embedded chatbot and bet the surface on agents I don't ship — anyone's agent, arriving over a protocol, greeted by tools instead of a UI. The WebMCP gateway predates the browser's own origin trial by five months.

## Track 5: "How the code knows itself" — six generations

This track carries the discipline that became its own post ([executable institutional knowledge](2026-07-12-executable-institutional-knowledge.md)).

| Gen | Born | What it was | Fate |
|---|---|---|---|
| 1 | Jan 25 | `.claude/index.json` plus an `index-maintainer` agent: a module catalog with exports, dependencies, and flow chains — four weeks into my coding life | Update cadence decayed from weekly to biweekly; deleted Apr 20, and the commit states the lesson: "drop stale index.json, generate block catalog at runtime" |
| 1.5 | Feb 18 | Handwritten `ARCHITECTURE.md` / `FEATURE-MAP.md` | Folded into the two-file pattern |
| 2 | Feb 27 | **20 Haiku agents**, each assigned a partition of the codebase, reading every file and writing dependency JSON — staged execution, random spot-checks against source | Superseded Apr 8 |
| 3 | Apr 8 | Deterministic parser replaced the agent fleet: stdlib `ast` for Python, a hand-rolled regex import-parser for TypeScript | Evolved |
| 4 | Apr 14 | Two-file pattern per domain: handwritten narrative + generated map | Alive |
| 5 | May 27–28 | The context overlay: per-file **and per-function** — doc links, dep-map partition, and embedding nearest-neighbors down to individual symbols (15,705 function-level vectors at the time), injected by a hook the moment an agent touches a file | Alive; June work pushed granularity toward per-statement |
| 6 | Jul 6–10 | Freshness monitors, self-healing cluster prune, auto-regeneration on integrator merge-drain | Alive |

The arc: maintained indexes rot at the speed of human attention — gen 1's update cadence visibly decays in the log before its deletion — so each generation moved further toward derived: regenerate on demand, then regenerate on event.

Gen 2 deserves its epitaph. For six weeks, the codebase's self-knowledge was twenty small language models reading source files and writing down what they saw. It worked. It also cost enough time and tokens that I went hunting for something faster — which is when I found out the rest of the world was converging on the same problem.

## Track 6: The agent org — from assistant to institution

| Date | Event |
|---|---|
| Jan 25 | First custom subagent definitions (`.claude/agents/`) |
| Feb 13 | Switched the in-app assistant to Haiku 4.5 and added rate-limit handling — the first model-tiering decision on record |
| **Mar 31** | **The intercom: 19 agents on a PostgreSQL-backed message bus** with push notifications. Later, when I went looking for a replacement, nothing better existed — so I kept it and hardened it |
| Apr 9 | Shared eval package |
| **Apr 21** | **The audit harness, in one day**: invariants, structural detectors, semantic (embedding) detectors, drift tracking |
| Apr 22 | "Rebuild audit-hardening work lost after session…" — the harness's first act was recovering its own lost construction |
| May 7–12 | The opus swarm: 394 commits in four rounds. On May 12 I wrote the agent-sizing skill — the paragraph I'd been retyping into prompts for weeks, finally codified |
| May 16 | `quarantine.py`: edits that bypass the pipeline get caught and held |
| **May 17** | **The integrator**: one agent owns main; all work arrives as gated branch submissions. I modeled it on what I thought a software team should look like, after agents kept overwriting each other's work — which is also why every agent works in its own clone. The same day, the original intercom relay was archived: the org structure retired the bus that built it |
| May 20–21 | Honesty-guardrail toolkit; the gate bounced a guardrails spec and I resubmitted it — the pipeline gating its own upgrades |

That last line is my favorite in the album. By late May, improvements to the system had to pass the system.

## The failure ledger

What died, and what each death taught:

- **`tradesite-simplified`** (Jan 17, one commit). Restarting clean feels like progress and usually isn't.
- **The in-site chatbot** (Feb 13 – Apr 16). A chat window bolted to an app is a demo, not a surface. I killed it deliberately, plumbing and all, for the agent-native bet.
- **DuckDB as the app database** (Jan 17 – Mar 15). A brilliant embedded analytics engine; the wrong tool under a concurrent web app. The Feb 23 double-attach errors were the tell.
- **The IBKR REST client** (Jan 25 – Mar 24). The socket client ran beside it in parallel-validation mode, then replaced it the same day — the first use of the run-both-compare-then-cut pattern that later became standard here.
- **`tradesite-worker`** (Mar 15 – Apr 8). A distributed compute worker with CLI hardware detection and TOTP enrollment. I built it in the SaaS-ambition sprint and removed it three weeks later when the GPU work moved in-house. The most fully-realized dead end in the repo.
- **The original `SuperCombo.tsx`** (Jan 6 – Feb 9). Survived one repo migration, not the overnight overhaul. Its name lives on, wrongly, in its replacement.
- **`index.json` + the index-maintainer** (Jan 25 – Apr 20). Death by staleness; the deleting commit states the lesson.
- **The Haiku mapping fleet** (Feb 27 – Apr 8). Twenty agents replaced by three hundred lines of `ast`. The boring solution ate the clever one.
- **The March 22–24 data loss.** A session's work — the IBKR socket client, database migrations, several features — was lost before it reached main. Six "recover" commits rebuilt it from two sources: eight files mined out of the session's own transcript ("Recovered from session transcript (3/22) — 8 files never committed") and a larger set salvaged from a dangling commit. Backups and WAL archival landed within 48 hours. The pipeline commits per merge now; we don't lose work anymore.
- **One repo-history rewrite** (Jun 29), with a punchline. The first attempt was gentle: a `.mailmap` merge to consolidate author identities, its commit message proudly noting "no history rewrite." The actual rewrite happened the same day — it orphaned the old SHAs, silently wiped months off my GitHub contribution graph, and made the "no history rewrite" commit its own first casualty. It survives only in a backup bundle, disconnected from main. Read the docs on how contribution credit works *before* rewriting history.

## The time machine

Seven months produced exactly one screenshot committed to git (February 4: the legal disclaimer, in its natural habitat, over the early strategy builder). For everything else I built a time machine: extract the frontend at a historical commit straight from the git object store, build it, serve it, visit it. The February 8 build compiled on the first try. When the reviewing agent tried to close the disclaimer without reading it, the resurrected app guilt-tripped it with a modal I'd completely forgotten writing — "TL;DR: It's worth the read" — then refused to enable "I Accept" until the text had actually been scrolled. Past me, gating present me's agents. And when the original SuperCombo finally rendered — 94 beginner blocks, empty canvas — it wore a red 401 banner, because July's backend refuses to serve data to an unauthenticated February ghost. Correct behavior, five months early.

## Epilogue

Read back, the history has three acts. Act one (Dec–Feb): I learned to code by shipping — features piled up, restarts happened, names stopped meaning things. Act two (Mar–Apr): the migrations — I replaced everything load-bearing at least once, and testing arrived with teeth. Act three (May–Jul): the institution — the agents got an org chart, a message bus, a gate, and an audit culture; throughput tripled; the failure ledger mostly stopped growing.

What I'd tell December-me: the product was never the strategy platform. It's the system that builds it.

---

*Reconstructed 2026-07-12 from `git log` across four repos. John Ward is a mechanical design engineer who fell into software in December 2025. Feedback welcome — file an issue on this repo.*
