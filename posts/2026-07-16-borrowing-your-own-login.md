# Borrowing your own login: scripting an SSO-gated enterprise API with zero stored credentials

*Published July 16, 2026. This is a how-to, not an invention claim — the ecosystem prior art is conceded up front, and per [my own audit standards](2026-07-16-auditing-my-own-origin-stories.md), the in-repo record is silent on who first proposed it here, so no origin story either. What follows is the technique, the constraints that select for it, and the sharp edges.*

The situation, genericized only slightly: our ERP (IFS Cloud) exposes an OData API behind corporate SSO. I needed scripted, repeatable, read-only pulls from it — inventory snapshots, transaction history, the raw material for reports the vendor UI can't produce. The textbook answers were all closed: no service account was going to be provisioned, no API credentials issued, no admin rights on the machine, no server to deploy to. The one authenticated thing in the building was the browser I was already logged into.

So the script borrows it.

## The mechanism

Chrome, started with `--remote-debugging-port=9222`, logged into the ERP normally. A Node script attaches over the Chrome DevTools Protocol, finds the logged-in tab, and evaluates a `fetch(url, { credentials: 'include' })` **inside the page context**. The OIDC session lives in that page; the fetch rides it. The response comes back over CDP to the script.

The properties that make this worth writing up:

- **Zero credentials at rest.** The script stores nothing, sees no password, holds no token. Auth lives where it already lived — in the human's SSO session — and expires when it expires.
- **Zero server-side footprint.** No integration user in the audit logs, no new principal to review. Every request is attributable to the person who logged in, because it literally is them.
- **The whole API surface, not a blessed subset.** Whatever the logged-in user can see in the UI, the script can pull as OData.

On top of that base, the pulls grew production manners: keyset pagination for large entity sets, and checkpointed, resumable runs — a multi-hour history pull survives the SSO session timing out mid-run; you re-login and it continues from the checkpoint instead of starting over. The helper began life inline in a query tool and was later extracted into a small reusable module, which is how you know a hack has become infrastructure.

## Prior art, conceded

Driving an authenticated browser session programmatically is what Puppeteer and Playwright *are* — Playwright's `APIRequestContext` and in-page `fetch` evaluation are documented mainstream features, and attaching to an already-running Chrome over CDP is documented too, including in the security literature, where "a debug port on a logged-in browser" is correctly described as an attack surface. I invented nothing here. What the constraints selected for, and what most write-ups don't cover, is the *attach-don't-launch* variant: not an automation-launched browser with provisioned credentials, but the human's own live session, chosen precisely because no credentials could ever be provisioned.

## The sharp edges, because they're real

The reason this technique appears in security advisories is the same reason it works: **the debug port is your session, unauthenticated, to anyone who can reach it.**

- The port binds to localhost; keep it that way, and treat anything already running on your machine as inside the trust boundary — because it is.
- Run the debug-port browser as a dedicated profile/instance for the purpose, not your everything browser with every tab of your life in it.
- This was done with permission, for read-only pulls, at my own employer, on data I already had UI access to. The technique doesn't grant access; it scripts access you already have. If it would grant access, you're on the wrong side of both the security model and your employment agreement.
- And the honest failure mode: it inherits the session's lifetime. Long pulls need the checkpointing above, or they die at token expiry — which in practice is the feature that forces you to build resumability, which you wanted anyway.

The general lesson travels beyond ERPs: when the auth path is closed and the human path is open, **the human path is an API** — attributable, permissioned, expiring, and already audited. Sometimes the enterprise-grade answer is the one that adds no new credentials at all.
