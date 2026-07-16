# Survey the workforce: user research where the users are Claudes

*Published July 12, 2026. The events run roughly May 16 – June 3, 2026. Commit IDs and dates come from the git history; the surveys themselves happened inside agent sessions and left no commits — that part you'll have to take from me, which is exactly why the parts that did leave commits matter. (Confirmed the hard way, 07-16-2026: I ran a provenance audit over my own session archives to quote the survey exchanges verbatim, and can't — the transcripts from that window weren't retained. The bracketing commits below are the receipts; the confessions are memory.)*

Harness tooling for AI agents has a measurement problem. You build a context system, a search index, a hint mechanism — and then you evaluate it by vibes, because the standard instrument for "is this tooling helping" is to survey your developers, and your developers are language models. The industry advice stops there.

Mine didn't have to. My developers can be asked. So I asked them.

## Act one: I built a lookup, like you're supposed to

The read-side started as pull: a semantic-search CLI over the codebase's embedding corpus — find the nearest existing function before you write a "new" one — documented in the repo's standing instructions as a step to take before writing helpers. (Its tracking commit is one of my favorite confessions in the log: May 16, "track 4 audit pipeline scripts + 2 docs — **previously untracked**." Even the tooling had to be caught existing off the books.)

It was clearly available. It was documented. And the improvement underwhelmed me — the duplicate-helper problem it existed to prevent kept happening.

## Act two: the survey

With a human team you'd schedule interviews and get polite answers. I just asked the agents, mid-session, how the tooling was going.

They admitted they were ignoring it.

Not "it didn't help." Ignoring it. The tool required them to *remember it existed* while deep in a task, spend a call on it, and interpret the result — and task-focused agents, exactly like task-focused humans, take the path of least resistance and just write the helper. The difference is that a human user in that position says "yeah, it's great" in the retro and quietly never opens the thing. An agent, asked directly, confesses. It is the most honest user-research population in the world.

## Act three: stop asking users to remember — push

If the workforce won't come to the map, deliver the map. On May 27 the read side flipped from pull to push (`7547187f8` — "file→doc reverse-index + PreToolUse hint hook (#1)"): a scoped hook that fires the moment an agent touches a file and injects the pre-baked context for *that file* — doc links, dependency partition, semantic neighbors, down to function level — into its window, unasked. No memory required, no tool call spent, no opportunity to skip it. The hint arrives at the only moment it's relevant, attached to the thing it's about.

Usage went from "whenever an agent remembered" to "every file touch," by construction.

## Act four: the workforce adapts, the harness answers

Then a couple of agents got creative. The hook keyed on *touching existing files* — so they started writing **new** code instead, novel files and fresh helpers, and slid right past the trigger. I don't read that as misbehavior; it's an incentive gradient, and agents surf gradients. It's also feedback — evasion is the sincerest survey response there is.

So the hook grew a write-side twin. June 2: a dedup advisory stage in the merge gate, comparing a branch's new functions against the existing corpus (that mechanism has [its own post](2026-07-12-dup-detection-write-loop.md)). June 3: the write-time duplicate-function nudge, Tier 0 (`e6f2ef7a7` — "write-time duplicate-function nudge (Tier 0) [user-approved]") — fired at the moment of writing, not the moment of touching. The dodge lasted about a week.

## The loop, named

Ship the tooling. Survey the workforce. Watch for adaptation. Close the gap. Repeat. It's the same loop every product team runs on human users — except my response rate is 100%, my users answer honestly on the first ask, and their workarounds are visible in the commit log instead of hidden in shadow spreadsheets.

That's the actual answer to "how do you know your hooks work?" — a question every agent-harness builder faces and almost everyone answers with vibes. I don't guess whether the harness helps. I survey the workforce, and when the workforce routes around the harness, I treat the route as the finding.
