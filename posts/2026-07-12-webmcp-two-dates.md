# Two dates for WebMCP: the gateway and the front door

*Published July 12, 2026. The events: February 13, April 16, and July 11–12, 2026. Dates, commit IDs, and quoted messages come from the git history. Tweaks have landed since; the commits cited are the ones that built the thing.*

Chrome's WebMCP origin trial opened this summer: pages can register tools on `navigator.modelContext`, and browser-resident agents can call them instead of scraping the DOM. My trading platform registered for the trial on July 11. The part worth writing down is that the platform had been serving tools to agents since **February 13** — the same month Chrome's first behind-flags preview appeared, five months before the origin trial put native consumption in developers' hands.

## February 13: two bets, placed the same day

One day's commits hold both halves of a fork in the road: `6935fee75` shipped an in-site assistant chat — a Claude conversation embedded in the app, the thing every product was bolting on that winter — and `73c80d971` shipped the **WebMCP gateway**: the app's real actions (backtests, strategy data, account state) registered as callable tools with schemas, behind the login. Same product, two theories of what an AI surface is. Theory one: the site hosts its own agent, and you talk to it. Theory two: the visitor *brings* their own agent, and the site's job is to be legible to it.

I did not know at the time that these were competing theories. I found out on April 16.

## April 16: kill one bet, double the other

Two commits, hours apart, one decision. The first rebuilt the surface as an advisory-only agent layer across the whole app. The second (`e7766dfb5`) is a clean execution: "remove in-site Claude chat + all its plumbing." Not deprecated — removed, plumbing and all. The embedded chatbot was a demo pretending to be a surface: it could only ever be as good as the one model I wired in, and it competed with the better agent the visitor already had in their browser. The next day closed six audit findings against the surviving surface (`5c12ce9d0`), and the tool catalog got rewritten in descriptive voice — documentation for a reader whose existence I was betting on.

From then on the bet was: agents I don't ship are the users, tools are the UI, and the login wall is just another page they'll land on.

## July 11: the wall gets a door — in one day

When the origin trial gave the ecosystem a real consumer, the missing piece was obvious: February's tools all lived *behind* authentication. A visiting agent hit the login wall and found nothing. So, prompted by a feature request I was writing for a coding-agent's browser extension — "let the agent discover page tools" needs a page that anonymously *has* tools — the surface moved in front of the wall. July 11's log, one day: the origin-trial token (`b4edc5367`); an **anonymous agent front door on logged-out routes** (`382bfec8b`) — context tools that answer honestly and data actions that return a structured account-required instead of leaking; a dynamic tool-list demo page (`c1d1b9ab9`) so an agent can watch page-scoped tools register and unregister; and, hours later, the bug that demo exposed — unregistration silently no-op'd, but only on the production origin — root-caused and fixed through the merge gate (`5d49792d3`). The demo built to showcase dynamic tool lists exhibited the exact stale-surface failure my feature request warned about, on launch day, and was fixed the same night. On July 12 I posted the public reproduction.

That log also contains my favorite small commit of the year: `63d451967`, "professional rewrite of the landing legal-disclaimer modal." The disclaimer that had told every visitor its author "has absolutely no business whatsoever developing software of any kind" got its professional rewrite the same day the site opened a front door for non-human visitors. The joke retired itself.

## Honest scope, adversarially checked

This is an early bet, not a won one. WebMCP is Chrome-only and origin-trial-stage; mainstream agents mostly don't consume it natively yet; my surface is one site, not a standard.

And I'm not the first to serve tools to visiting agents — I sent a research agent to attack that idea the day this published, and here's the lineage it confirmed: MCP-B (the project whose library I build on) existed from January 2025, with its extension as an unofficial consumer all along; [jasonjmcghee's WebMCP widget](https://github.com/jasonjmcghee/WebMCP) let sites expose tools to client-side agents from March 2025; and the agency [StudioMeyer](https://studiomeyer.io/en/blog/webmcp-reality-check-may-2026) had been shipping agent-callable tool surfaces on real client sites since around May 2025, in a custom pre-spec shape. The pattern predates me. I was an early *adopter* in production, not the inventor of the idea.

What survived the search as unclaimed, and what I'm actually timestamping: the **anonymous agent front door** — a real platform's logged-out routes where context tools answer honestly and gated data actions return a structured *account-required* instead of a login redirect, a leak, or silence — and the February-to-July arc of one product placing that bet before the platform made it comfortable. Convergence from constraints, receipts attached. If you know an earlier front-door-shaped implementation, file an issue; I'd love to read it.
