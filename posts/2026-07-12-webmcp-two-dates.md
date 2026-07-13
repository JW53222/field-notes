# Two dates for WebMCP: the gateway and the front door

*Published July 12, 2026. The events: February 13, April 16, and July 11–12, 2026. Dates, commit IDs, and quoted messages come from the git history. Tweaks have landed since; the commits cited are the ones that built the thing.*

Chrome's WebMCP origin trial opened this summer: pages can register tools on `navigator.modelContext`, and browser-resident agents can call them instead of scraping the DOM. My trading platform registered for the trial on July 11. The part worth writing down is that the platform had been serving tools to agents since **February 13** — five months before the browser had an official way to consume them.

## February 13: two bets, placed the same day

One day's commits hold both halves of a fork in the road: `6935fee75` shipped an in-site assistant chat — a Claude conversation embedded in the app, the thing every product was bolting on that winter — and `73c80d971` shipped the **WebMCP gateway**: the app's real actions (backtests, strategy data, account state) registered as callable tools with schemas, behind the login. Same product, two theories of what an AI surface is. Theory one: the site hosts its own agent, and you talk to it. Theory two: the visitor *brings* their own agent, and the site's job is to be legible to it.

I did not know at the time that these were competing theories. I found out on April 16.

## April 16: kill one bet, double the other

Two commits, hours apart, one decision. The first rebuilt the surface as an advisory-only agent layer across the whole app. The second (`e7766dfb5`) is a clean execution: "remove in-site Claude chat + all its plumbing." Not deprecated — removed, plumbing and all. The embedded chatbot was a demo pretending to be a surface: it could only ever be as good as the one model I wired in, and it competed with the better agent the visitor already had in their browser. The next day closed six audit findings against the surviving surface (`5c12ce9d0`), and the tool catalog got rewritten in descriptive voice — documentation for a reader whose existence I was betting on.

From then on the bet was: agents I don't ship are the users, tools are the UI, and the login wall is just another page they'll land on.

## July 11: the wall gets a door — in one day

When the origin trial gave the ecosystem a real consumer, the missing piece was obvious: February's tools all lived *behind* authentication. A visiting agent hit the login wall and found nothing. So, prompted by a feature request I was writing for a coding-agent's browser extension — "let the agent discover page tools" needs a page that anonymously *has* tools — the surface moved in front of the wall. July 11's log, one day: the origin-trial token (`b4edc5367`); an **anonymous agent front door on logged-out routes** (`382bfec8b`) — context tools that answer honestly and data actions that return a structured account-required instead of leaking; a dynamic tool-list demo page (`c1d1b9ab9`) so an agent can watch page-scoped tools register and unregister; and, hours later, the bug that demo exposed — unregistration silently no-op'd, but only on the production origin — root-caused and fixed through the merge gate (`5d49792d3`). The demo built to showcase dynamic tool lists exhibited the exact stale-surface failure my feature request warned about, on launch day, and was fixed the same night. On July 12 I posted the public reproduction.

That log also contains my favorite small commit of the year: `63d451967`, "professional rewrite of the landing legal-disclaimer modal." The disclaimer that had told every visitor its author "has absolutely no business whatsoever developing software of any kind" got its professional rewrite the same day the site opened a front door for non-human visitors. The joke retired itself.

## Honest scope

This is an early bet, not a won one. WebMCP is Chrome-only and origin-trial-stage; mainstream agents mostly don't consume it natively yet; my surface is one site, not a standard. The claim I'm timestamping is narrower and, I think, more interesting: a solo builder, seven weeks into software at the time, shipped an agent-native tool surface in February 2026 because the login page kept being useless to the agents visiting it — and the browser platform arrived at the same shape five months later. Convergence from constraints, receipts attached. If you know earlier prior art for a production site treating visiting agents as first-class users via `modelContext`-style tools, file an issue; I'd love to read it.
