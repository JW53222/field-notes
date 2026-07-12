# Executable institutional knowledge: regression-testing a system you can't read

I'm a mechanical design engineer at a make-to-order manufacturer. I opened VS Code for the first time last December. Since then I've built a browser extension that six departments at my plant use daily to get sane answers out of our ERP — and this post is about the one practice from that project I think other people should steal.

## The problem: discoveries about someone else's system

Our ERP (IFS Cloud) was set up years ago in a configuration that doesn't match how we actually operate. The documentation for *our* instance effectively doesn't exist. So every useful fact I know about it was **discovered**, not read: probed out of the OData API, cross-checked against paper travelers on the shop floor, or learned the expensive way when a report was wrong in front of people who needed it to be right.

Here's the thing about that kind of knowledge: it has a half-life. It lives in one person's head, or in a comment nobody reads, or in a Slack thread nobody can find. Six months later somebody — a new hire, a contractor, *you on a bad day*, or increasingly your AI coding assistant — rediscovers the plausible-but-wrong way to do it, and you pay for the same lesson twice.

You can't unit-test the vendor's code. You can't read their source. But you **can** test your beliefs about their system, every time your CI runs.

## The pattern: invariants with provenance

My repo carries a file of invariants — currently 31 of them. Each one is a claim about the external system, written in positive form, with a link to the evidence, a machine-checkable rule, and a date. A trimmed real example:

```json
"no-banned-revision-fields": {
  "claim": "Production code reading from ShopOrds or EngPartMasters must not use
            EngChgLevel/ValidRevision/PartRevision. These fields always return '1'
            on those entities — use Cf_C_Revision_No (ShopOrds) or LastRevision
            (EngPartMasters). EngChgLevel is legitimate on ProductStructure (it is
            the canonical key field on that entity).",
  "severity": "high",
  "finders":  [{ "kind": "grep",
                 "pattern": "\\b(EngChgLevel|ValidRevision|PartRevision)\\b" }],
  "exceptions": [],
  "evidence": { "docs": ["docs/business-definitions.md"] }
}
```

That claim cost real time to establish. Three different "revision" fields, all of which *look* right, all of which silently return `'1'` on the entities we query — except on one entity where the same field is the canonical key. No error, no warning, just wrong revisions on drawings. Now it's a grep pattern. If anyone (human or AI) reaches for the plausible field again, the build goes red and the failure message *is the documentation*.

A few more, so you get the flavor of what earns a slot:

- **`objstate-uses-enum-form`** — every `Objstate eq` filter must use the fully-qualified enum form `IfsApp.{Projection}.{EnumType}'Value'`. Plain strings always 400 from IFS Cloud. The checker just looks for the qualified form within three lines of the finder hit.
- **`no-datetime-literal`** — OData v2 `datetime'...'` literals always 400 against this OData v4 API. Copy-pasting old OData examples from the internet is exactly how this one kept reappearing.
- **`no-divide-time-fields-by-60`** — the operation time fields (`MachSetupTime`, `LaborRunTime`, …) are **already in hours**. Divide by 60 like they look like they want you to and you understate labor by 60x. The repo has zero hits; the rule exists purely as a tripwire, because this mistake was made once and it was expensive.
- **`overdue-uses-planned-ship-date`** — a pure business rule: "overdue" at our shop keys on `PlannedShipDate`, not the two other delivery-date fields that seem equally plausible. The claim carries a name and a date (`Ward 2026-06-03`) and a pointer to the business-definitions doc where the reasoning lives.
- **`no-zero-rate-fallback`** — a `|| 0` fallback on a labor-rate lookup is forbidden unless an explicit zero-rated-department skip guard sits within six lines. Zero-rating an unmapped department doesn't crash anything; it just quietly understates cost. The finder is deliberately over-broad — for rules like this, false positives beat missed bugs.

## The rules of the discipline

The machinery is almost embarrassingly simple — a grep finder plus an optional "this other pattern must appear within N lines" checker, a couple hundred lines of runner. Semgrep could do it. That's not the point. The point is the contract each entry has to honor:

1. **The claim is written in positive form.** Not "don't use EngChgLevel" but "these fields return '1' on these entities; use these instead; here's the one entity where the field is legitimate." The rule teaches the fix, not just the prohibition.
2. **Every claim carries provenance.** A doc link, a name, a date. When someone challenges the rule — and they should — the argument ends with a link instead of a memory contest.
3. **Exceptions are dated and reviewed.** An exception older than 180 days gets flagged for re-review. Grandfathered escapes are how rule systems rot.
4. **Zero evaluations is yellow, not green.** If a rule's finder matched nothing anywhere, that's reported as "this rule didn't exercise," not silently passed. A rule that never fires might be protecting nothing — or might be misspelled.
5. **A discovery isn't done until it's a rule.** The debugging session that established the fact ends by writing the invariant. That's the actual discipline; everything else is tooling.

## Why this matters more with AI assistants

I write most of my code with an AI assistant, and I'll say the quiet part: assistants are *fantastic* at confidently producing the plausible-but-wrong version of vendor API usage. They've read the whole internet's OData v2 examples. They have not debugged *your* instance. The invariants file is how discoveries survive between sessions — the assistant edits code, the tripwires fire, and the failure message teaches it (and me) the local truth. It has become the single highest-leverage file in the repo for keeping AI-generated code honest against a system neither of us can read the source of.

## What it costs and what it buys

Costs: writing the rule while the discovery is fresh (ten minutes if you do it immediately, a day of re-derivation if you don't), and tending the occasional false positive.

Buys: arguments that end with links. Onboarding — human or AI — that reads claims instead of folklore. A build that fails when the mistake you paid for in June tries to come back in November. And something harder to name: the file *is* the institution's knowledge, in a form that can't quietly become a lie, because it runs.

## How to start

1. Next time you discover a non-obvious fact about a system you don't control, write it down as a positive-form claim with a date and your name.
2. Write the dumbest grep that would catch the mistake the claim prevents. Dumb is fine. Over-broad is fine for high-severity rules.
3. Put it in CI. Make zero-matches visible as "didn't exercise," not silent green.
4. Date every exception. Review old ones.
5. End every painful debugging session by asking: *what rule would have made this impossible?* Write that rule while it hurts.

Thirty-one rules after six months. Each one is a scar with a regex on it. I'd rather have the file than the memories.

---

*John Ward is a mechanical design engineer who fell into software in December 2025. Feedback welcome — file an issue on this repo.*
