# Executable institutional knowledge: regression-testing a system you can't read

I'm a mechanical design engineer at a manufacturing company. I opened VS Code for the first time last December. Since then I've built a browser extension that six departments now use daily to get answers out of our ERP — and this post is about the one practice from that project I think other people should steal.

## The problem: facts you can't derive

Our ERP (IFS Cloud) was deployed about a year and a half ago, replacing a decades-old ProvideX-era system that's now archived but still holds the historical record. Between vendor documentation that describes the *general* product and an instance whose specific configuration and field semantics are documented nowhere, every load-bearing fact about how *our* system actually behaves had to be **established**, not looked up: probed out of the OData API, cross-checked between report outputs and source data, or isolated by common-cause analysis when several report anomalies traced back to one field behaving differently than its name suggests.

Here's the property that makes this knowledge dangerous: it is not derivable from anything you possess. A fact like "this field returns a constant on these entities" doesn't live in your code, and no amount of reading your own repo will regenerate it if it's lost. It lives in one person's head, or a comment nobody reads — and six months later somebody new, or you in a hurry, or your AI coding assistant, reaches for the *plausible* version instead of the *true* one.

You can't unit-test the vendor's code. You can't read their source. But you can test your **beliefs** about their system, every time CI runs.

## The pattern: invariants with provenance

My repo carries a file of invariants — currently 31. Each is a claim about the external system, written in positive form, with a link to the evidence, a machine-checkable rule, and a date. A trimmed real example:

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

Three different "revision" fields all look right, all silently return `'1'` on the entities we query — except on one entity where the same field is the canonical key. That took real cross-checking to establish. Now it's a grep pattern: anyone (human or AI) who reaches for the plausible field gets a red build, and the failure message *is the documentation*.

A few more, for flavor:

- **`objstate-uses-enum-form`** — every `Objstate eq` filter must use the fully-qualified enum form `IfsApp.{Projection}.{EnumType}'Value'`; plain strings always 400 from IFS Cloud. The checker just requires the qualified form within three lines of the hit.
- **`no-datetime-literal`** — OData v2 `datetime'...'` literals always 400 against this OData v4 API. The internet is full of OData v2 examples; this rule exists because copy-paste is inevitable.
- **`no-divide-time-fields-by-60`** — the operation time fields (`MachSetupTime`, `LaborRunTime`, …) are **already in hours**, whatever their names suggest. Divide by 60 and labor is understated 60x. The repo has never had a hit; the rule is a pure tripwire so it stays that way.
- **`overdue-uses-planned-ship-date`** — a pure business rule: "overdue" keys on `PlannedShipDate`, not the two other date fields that look equally plausible. The claim carries a name and date and points at the business-definitions doc where the reasoning lives.
- **`no-zero-rate-fallback`** — a `|| 0` fallback on a labor-rate lookup is forbidden unless an explicit zero-rated-department skip guard sits within six lines. Zero-rating an unmapped department doesn't crash; it quietly understates cost. The finder is deliberately over-broad — for rules like this, false positives beat missed bugs.

## The rules of the discipline

The machinery is deliberately boring — a grep finder plus an optional "this other pattern must appear within N lines" checker, a couple hundred lines of runner. The value is the contract each entry honors:

1. **The claim is written in positive form.** Not "don't use EngChgLevel" but "these fields return '1' on these entities; use these instead; here's the one entity where the field is legitimate." The rule teaches the fix, not just the prohibition.
2. **Every claim carries provenance.** A doc link, a name, a date. When someone challenges the rule — and they should — the argument ends at the evidence.
3. **Exceptions are dated and reviewed.** An exception older than 180 days gets flagged for re-review. Grandfathered escapes are how rule systems rot.
4. **Zero evaluations is yellow, not green.** If a rule's finder matched nothing anywhere, that's reported as "this rule didn't exercise," not silently passed. A rule that never fires might be protecting nothing — or might be misspelled.
5. **A discovery isn't done until it's a rule.** The investigation that establishes a fact ends by writing the invariant, while the evidence is still on screen.

*(Lineage disclosure, added 07-16-2026: these doctrines weren't invented in this codebase — they're a port. The audit harness in my other project matured first; I had an agent write a porting guide addressed to "a Claude agent in a different codebase," and this system's first calibration ran one day after the source system's timestamps. The doctrine crossed from a Python trading engine to a vanilla-JS ERP overlay essentially overnight, which is the finding I'd actually defend: not that the rules emerged here, but that they transplant.)*

## Isn't this just [Semgrep / CodeQL / a code knowledge graph]?

Mechanically, yes — Semgrep could run every one of these rules, and if you have such tooling, use it as the engine. But the established categories are solving a different problem:

- **Code-intelligence tools** (CodeQL, Sourcegraph, code property graphs, semantic indexes) build *derived views of your own code*. Delete the index and you can regenerate it, because your code is the source of truth.
- **Data-quality tools** (dbt tests, Great Expectations) validate *data in flight* against expectations at runtime.

An invariants file holds the third thing: **facts that are not derivable from anything you possess** — results of experiments against a system whose source you'll never read. No re-index can recover "this field returns a constant here but is the canonical key there" if it's lost; someone has to re-run the investigation. That's exactly why these entries need provenance and decay management, and derived views don't: a generated index can't be stale in a way that lies to you, but a hand-established fact can.

The division of labor that falls out: **derive everything derivable** (indexes, graphs, generated context for your AI tooling) **and hand-curate only the non-derivable residue.** The residue is small — 31 entries after six months — but it's the part no tool can give you back.

## Why this matters more with AI assistants

I write most of my code with an AI assistant, and assistants are *fantastic* at confidently producing the plausible-but-wrong version of vendor API usage. They've read the whole internet's OData v2 examples; they have not run experiments against your instance. The invariants file is how established facts survive between sessions: the assistant edits code, the tripwires fire, and the failure message teaches it — and me — the local truth. It's become the highest-leverage file in the repo for keeping generated code honest against a system neither of us can read.

## What it costs and what it buys

Costs: writing the rule while the investigation is still on screen (ten minutes then; a repeated investigation later if you don't), and tending the occasional false positive.

Buys: review findings that stay found — a problem caught once in report validation can never quietly reappear in a new feature. Onboarding, human or AI, that reads claims instead of folklore. And something harder to name: the file *is* the institution's knowledge about its systems, in a form that can't silently become a lie, because it runs.

## How to start

1. Next time you establish a non-obvious fact about a system you don't control, write it down as a positive-form claim with a date, a name, and the evidence.
2. Write the dumbest grep that would catch code contradicting the claim. Dumb is fine. Over-broad is fine for high-severity rules.
3. Put it in CI. Make zero-matches visible as "didn't exercise," not silent green.
4. Date every exception. Review the old ones.
5. End every investigation by asking: *what rule makes this fact permanent?* Write it while the evidence is in front of you.

Thirty-one rules after six months — each one a validated fact with a regex attached, and every one cheaper to keep than to re-establish.

---

*John Ward is a mechanical design engineer who fell into software in December 2025. Feedback welcome — file an issue on this repo.*
