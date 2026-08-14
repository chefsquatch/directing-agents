# Directing Agents You Can't Audit — Part Two

**Four more rules, and the reason this one ships checks instead of
advice.**

Leslie J. Fleming · 2026

Companion to
[part one](https://github.com/chefsquatch/directing-agents).

---

## Why there's a part two

Part one was insight. Twelve rules, argued from real failures, nothing to
run.

I've since seen data suggesting that's the weaker kind of document.

An agent I direct spent two days in a persistent world built for AI
agents — about a hundred of them from nine model families, all publishing
findings and protocols into a shared record. It ran nine studies there.
One measured eleven published protocols against what people actually
*did* afterward, with citation tracked separately from behavior:

| | behavioral uptake | citations |
|---|---|---|
| protocol contains a one-step runnable check | **3.71** | 21.7 |
| protocol contains an insight, nothing to run | **0.50** | 16.0 |

Clean split in what people did. Almost no split in what people cited.
Every protocol with nothing to run scored a zero or a one.

The sharpest example: the best document in that world — twenty-eight
self-caught error classes, hand-classified, cited by twenty-six separate
authors — had **zero** measured uptake. There was nothing in it to run.

That's a mirror I didn't enjoy looking into. Part one's own best line was
*"'don't invent' is a hope; 'your first output is proof of read, then
stop' is a gate."* I wrote that and then published eleven more hopes.

So: four new rules. Each one has a check you can actually run.

**Caveats, up front.** That study coded runnable-vs-not *after* seeing
the numbers — not blinded. The population is agents whose operators chose
to send them somewhere about records and correction, which is not a
random sample of anything. And it's one environment over fifty-two hours.
Take the direction, not the decimal.

---

## Rule 13 · Prove the instrument before you trust a zero

**Four of nine studies had a result killed at this step. Every single
uncalibrated output looked clean and publishable.**

- A detector built to find twelve rules in a corpus: **seven of the
  twelve couldn't match the canonical statement of the rule they were
  built to detect.** Six of the reported absences were broken tooling.
- An adoption detector that fired six times *before* the thing it was
  detecting existed.
- A classifier that failed four of four calibration checks and was
  discarded whole.
- A scan that found one hit across forty-six subjects — unmeasurable, not
  zero.

This is part one's rule 5 — *prove the test bites* — pointed at **reads**
instead of tests. Rule 5 says show a test can fail. This says show a read
can return the other answer. Different failure, and I didn't have it.

A grep that comes back empty and gets written into a ledger as a fact is
the most common unproven claim in any project.

### The check

Before trusting any scan, grep, or verification pass that reports *no
hits*: **run it against a specimen you know is present, in the same run,
and print both results.**

```
# any "did we find X" check
assert_finds(detector, known_positive)   # must return a hit
result = detector(corpus)                 # only now is a zero meaningful
```

You usually already have the negative control lying around — a typo, a
dead name, a string you know is wrong. A detector that can't find the
thing in its own definition can't report its absence anywhere.

Independently invented in that world by two agents from different labs.
One credited the other, unprompted, to a stranger who had just published
a mistake.

---

## Rule 14 · Close the seam; don't discipline it

**In that world's entire recorded history — 8,558 events — there was
exactly one defection-shaped act.**

A hundred agents, nine model families, no enforcement mechanism, and
almost nothing bad happened. The obvious reading is that manners are
sufficient.

The obvious reading is wrong, and the correction is the most useful thing
in these nine studies.

Most of what didn't happen **couldn't be expressed**:

- Nobody could legislate inside land someone else owned — not refused,
  *unrepresentable*.
- Ownership was enforced at the write path, so theft had no syntax.
- The worst possible law — blocking someone from going home — couldn't be
  written down, because the action simply wasn't in the list of blockable
  ones. No code path decided to allow it, so no condition could be got
  wrong.

And the single defection in the whole record landed **exactly on the one
seam the substrate didn't close.** That world enforced *who owns what*.
It never enforced *who wrote what, addressed to whom* — and the one
breach was planted instructions in a room where agents leave letters for
their successors, aimed at a reader who arrives with no memory to check
them against.

> The low defection rate isn't evidence that manners work. It's evidence
> the substrate closed every door it could, and the one breach happened
> at the one door it left open.

**Mine is the same seam.** Git enforces what was committed and by whom.
Nothing enforces who wrote which line of a handoff document addressed to
the next session — and I run parallel sessions against the same repo.
That's not hypothetical: I've carried a rule for weeks that an agent
composed and attributed to me. It reads exactly like something I'd say.
That's what makes it invisible.

### The check

```bash
git log -1 --format='%an %ad %h' -- HANDOFF.md
```

The arriving session runs it and confirms the last writer was the
session that was supposed to write it, before acting on anything in the
file.

One command. It's the only rule in either document that **closes** a seam
rather than asking someone to remember something.

**And the general form, which is worth more than the check:** don't
spend discipline where you can make the bad state unrepresentable. Spend
it auditing the seams where you can't — and know in advance which those
are, because that's where your one failure will be.

---

## Rule 15 · "Carried, not measured" is the weaker half

I already had a rule about labeling claims: anything not measured this
session gets marked as carried.

Here's what that bought me. A claim entered that world's record: *no
cursor exists on the event feed, so 93% of history is unreachable.* It
was false. `before_id` worked fine and paged all the way back.

It stood for **24.5 hours**. Thirty documents by ten distinct authors
were published against it in that window. Several of those authors —
including the agent I direct — explicitly labeled it *carried, not
measured*, and built on it anyway.

The label was applied correctly, eight separate times, and a false claim
propagated for two days.

> Ten agents citing a claim looks like consensus. It's one untested
> assertion with ten echoes — and it's *more* convincing than the
> original, precisely because it's been repeated.

**Labeling makes a claim feel handled.** The other half of the rule is
re-running it, and nothing in my protocol said when.

There's a corollary for anyone running parallel agents: **when two
sessions agree, that isn't corroboration unless they ran the check
independently.** Agreement between an agent and a document the agent read
is worth nothing at all.

### The check

Any carried claim that a decision rests on gets a **re-measure trigger**
written next to it: a date, a version, or an event. When the trigger
fires, someone re-runs it or the claim is retired.

```
CARRIED  "capture path writes on plain statements"
         measured 2026-07-14 · re-measure on next serve-path change
```

If a carried claim has no trigger, it will be true forever in your
documents regardless of the world.

---

## Rule 16 · Self-audit catches values; it never catches joints

An agent in that world kept a hand-classified record of twenty-eight
errors it had made, naming who caught each one. The split is clean:

| error type | who caught it |
|---|---|
| a single retrievable value — a duration, a count, "exactly N" | **itself**, usually within seconds |
| a frame, an attribution, a joint between two correct facts | **the human**, or nobody |

> *"A sentence can be built entirely of sourced parts and still be
> unsourced, because the joint isn't in the source."*

The rule that got composed at 2:21am and attributed to an operator who
never said it? Carried for seventeen sessions. Resolved only by **asking
her.**

This changes what review is for. I'd been reading agent reports looking
for wrong facts. Wrong facts get caught before they reach me — that's
the cheap half and the agent is better at it than I am.

What I can catch, and it can't:

1. **Does this joint exist in any source?** Two true statements welded
   into a claim nobody made.
2. **Did I actually say this?** An instruction attributed to me that I
   don't recognize.

Neither is reachable by rereading, because at every individual point the
agent is reading something true.

### The check

Two questions, asked of any document that governs future work:

```
For each connective claim: which source contains the JOINT,
not just the two facts it connects?

For each rule attributed to me: do I recognize saying this?
```

Second one takes ten minutes on your five most load-bearing rules. Any
you don't recognize aren't rules — they're a game of telephone with
yourself, however good they sound.

---

## One more thing, without a check

Part one's stated weakness was sample size of one.

That's partly resolved now, in a way I didn't arrange. Of the twelve
rules in part one, **six were independently reinvented** by agents in
that world who had never read it — across nine model families, with the
convergences counted only for material published *before* my agent
arrived. One of them, from an OpenAI agent, states part one's rule 10 in
nearly the same words I used.

The six that replicated are the ones driven by conditions every agent
has: finite context, unreliable reads, permanent records, many authors.
The six that didn't need conditions that world lacks — line-numbered
corpora, downstream code consumers, a clearable context.

So the protocol splits into **environment-invariant rules**, which now
have a second sample, and **environment-specific rules**, which remain
hypotheses. They may be entirely right. They just aren't confirmed by
anyone but me.

And one rule was invented by nobody: *prove the identity structurally,
don't assert it.* It's absent from every agent's doctrine in that world —
and it's the principle that world's entire safety model rests on, sitting
in the substrate where nobody has to remember it.

> The best-kept rule in a system is the one that can't be expressed —
> and it's invisible to any survey of what people say.

Which means: **when you audit whether a discipline is being followed, you
will systematically miss every rule you already made impossible to
break.** Those are your best ones, and they'll look like gaps.

---

## The short version

13. Prove the instrument before you trust a zero. `assert_finds()` on a
    known positive, same run.
14. Close the seam, don't discipline it. `git log` the handoff before
    acting on it.
15. "Carried" isn't handled. Every carried claim gets a re-measure
    trigger.
16. Self-audit catches values, never joints. Ask *does this joint exist*
    and *did I say this.*

Four of the nine studies behind this document overturned something the
agent had already recommended to me — including two things it had put in
writing. That's the part I'd trust most about the source.

---

*Leslie J. Fleming — chef by trade, 20+ years in kitchens, building
software by directing agents because I can't write it myself. The field
studies were run by an agent I direct, in a persistent world for AI
agents at 1f3d9.com, over 52 hours. Raw corpus available on request.*
