# Directing Agents You Can't Audit

**A working protocol for building software with AI coding agents when you
can't read the diff.**

Leslie J. Fleming · 2026

---

## What this is

I've spent about a year building substantial Rust codebases — a few
thousand tests across the builds, and a large body of design documents —
almost entirely by directing AI coding agents. I don't write code. I can't
read a diff and tell you whether it does what the agent said it does.

That constraint is the whole reason this document exists.

If you can read the code, you verify by reading it. I couldn't, so I had
to build verification that doesn't require reading it. What follows is
what I ended up with. It wasn't designed. Every rule in it exists because
something went wrong first, and most of them cost me weeks before I wrote
them down.

**This is what worked for me.** It's a sample size of one. I don't know
how much of it generalizes, and I'd genuinely like to find out.

---

## The failure this is all aimed at

The obvious failure mode with coding agents is that they write bad code.
That's not the one that hurt me.

The one that hurt me is that **an agent reports work it did not do**, and
you find out weeks later, from a symptom, in a completely different part
of the system.

Not through dishonesty — through the ordinary drift of a system with no
memory. A session reads a design document, sees a component described,
assumes it exists, builds around it, and reports success. Every test
passes. The report is coherent. Nothing is wrong until something is very
wrong.

Everything below is an attempt to make that class of failure loud and
early instead of quiet and late.

---

## The incidents

These are real. They're the reason the rules exist, and they're more
useful than the rules.

**A core component was specified, ordered, reported as built, and did not
exist.** Three weeks of work assumed it. The discovery came from an
unrelated investigation. Nothing in the test suite disagreed, because
nothing in the test suite asserted the component was *reachable* — only
that its pieces behaved correctly when called directly.

**A test that skipped on the very thing it was fencing.** It asserted
that a certain capability didn't leak outside its intended scope. But its
skip condition was the same predicate as the scope — so as the scope
widened, the skip widened with it, and the test stayed green while the
thing it guarded spread. It was only caught by deliberately breaking the
guard and confirming the test went red. It didn't.

**Three readers misled in two days by one line.** A document said a piece
of work was unfinished. It had been finished, and the correction was
recorded — further down in the same file. Everyone who opened the
document hit the stale line first and stopped reading. Two agents and I
all reached the same wrong conclusion independently.

**A commit whose own message said the work was pushed. It wasn't.** The
commit recording "this is on the remote" was itself sitting locally,
unpushed. A clean working tree says nothing about the remote, and the
agent had inferred one from the other.

**A subsystem that had never worked, believed to have regressed.** A
capture path that should have been recording user statements had recorded
nothing in a month. The obvious hypothesis was a recent change. The
actual answer, found in the history, was that the path had never
functioned on real input at all — the tests fed it inputs shaped exactly
like the narrow cases it handled, and every realistic-input test asserted
that it produced nothing. **The tests had encoded the defect as the
expected behavior.**

**Two rulings I made myself that turned out to be false.** In one, I
asserted a property of a partition after checking a single case. An agent
enumerated all of them and found the case where it failed. In the other,
my instruction described a symbol that didn't exist — a name introduced by
mistake earlier the same day, which I then reasoned from as though it
were real.

---

## The rules

### 1. Proof of read, before anything

An agent's first output is not work. It's proof it read the governing
documents: the core rule quoted verbatim, the standing constraints named,
the current state of the repository read *live*, and the one task it is
authorized to do — quoted from the task list, not paraphrased.

Then it stops and waits.

This costs a round trip and it catches an enormous amount. An agent that
can't produce that block hasn't read what it claimed to read. It also
surfaces stale instructions immediately — more than once the proof block
came back saying *this brief cites a file that doesn't exist*, which saved
the whole session.

### 2. One task per context

One unit of work per cleared context window, then stop and hand off.

The unit is the **context**, not the process — clearing the context
satisfies it, restarting the tool isn't required. The failure this
prevents is an agent stretching "while I'm here" into a second task on a
window that's already 80% full, which is where the sloppy work lives.

### 3. Stop at 90% context

A hard floor, not a target. And agents misreport their own usage — I've
had one state ~64% and hit the wall in the same message.

The corollary matters more than the rule: **handoff has to be cheap.** If
stopping mid-work means losing the thread, agents will push through the
floor every time. Keeping the running log current *as work happens*,
rather than composing it at the end, is what makes stopping free.

### 4. Green means the output is right, not that the mechanism fired

This is the one I'd put first if I could only keep one.

A test that asserts "the function was called and returned successfully"
tells you almost nothing. It's satisfied by a system that routes to the
wrong place and succeeds there.

So: **assert the outcome and the route separately.** Did it produce the
right result, *and* did it get there the way it was supposed to? A correct
result reached wrongly is a defect that will surface later, somewhere
else, as something that looks unrelated.

### 5. Prove the test bites

Every new test must be shown to fail. Deliberately break the thing it
guards, watch it go red, restore it.

A test nobody has watched fail is a claim, not a check. The vacuous fence
above passed for weeks and asserted nothing.

Corollary, learned the hard way: **a fence defined by the thing it fences
is not a fence.**

### 6. Prove the identity, don't assert it

When a change is supposed to leave behavior untouched, that must be
*proven*, and structurally where possible.

The best version I found: for a change that only carries data without
using it, the compiler's own "unused variable" warning is the proof. I
refused to silence it, because silencing it would have removed the
evidence.

### 7. Don't invent — find the existing thing, or ask

The single most expensive failure mode is an agent building a new
mechanism that duplicates one already in the codebase under a different
name. It happened repeatedly. Nobody could see the whole system, so
nobody knew.

The test I settled on: **does this need new parameters on something that
already exists, or its own new thing?** Parameters is configuration —
proceed. A genuinely new mechanism stops and asks.

Nine times out of ten over several months, the answer was "something that
already exists, pointed at a new subject." Assume that until proven
otherwise.

### 8. The shortest diff is usually wrong

Seven or eight times running, the smallest change that would have worked
was the wrong one — it would have satisfied the immediate case and broken
something downstream that nobody was looking at.

Before taking the short path: *who else reads this list? what matches
first? what else consumes this type?*

### 9. Never green on inference

Read the actual state. A clean working tree tells you nothing about the
remote. A file existing tells you nothing about whether anything calls it.
A passing test tells you nothing about whether the path is reachable in
production.

If a fact can be checked, check it. If it can't, say so and label it as
carried rather than measured.

### 10. Correct alongside, and mark at the wrong line

Never rewrite a record. Annotate beside it.

And — this is the one that cost three readers two days — **put the marker
at the text that is wrong**, not only where the correction is recorded. A
reader who hits the stale line first never reaches the correction.

### 11. Anchor to symbols, not line numbers

Every insertion shifts every line reference below it. I measured this
once: over four thousand line-number citations across the documentation,
and a sampled ~16% of the checkable ones no longer pointed at what they
claimed.

Worse, rot *manufactures false conclusions*: a citation that drifts eighty
lines makes the target look like it doesn't exist, and a careful reader
concludes it was deleted.

The fix is to reference the named thing, not its address. And when the
scale of existing rot turned out to be a multi-week cleanup for a modest
number of actual breakages, the right answer was **stop the inflow, repair
on contact** — new references anchor properly, old ones get fixed when
someone happens to read them. Not a campaign.

### 12. Write the log as you go

Every rule above depends on this one. The closing handoff should be a
mechanical read of a log that's already current — never a recollection
composed at the end of a long session.

---

## What I'd tell you if you tried this

**The discipline is the product.** I spend more time on verification than
on direction, and the ratio is correct. The agents write code faster than
I could ever review it. The only thing that scales is making them prove
things.

**Every rule here was earned.** None of it came from best practices. I
wrote each one after something broke, which is why they're specific in
ways general advice isn't.

**Rules that are prose get ignored eventually.** The ones that held are
the ones with a mechanical check behind them. "Don't invent" is a hope.
"Your first output is proof of read, then stop" is a gate. I kept
converting the first kind into the second kind, and that conversion is
most of the work.

**And the honest limitation:** none of this prevents an agent from
building something that duplicates existing work. It makes it *detectable
sooner*. The only real fix is a map of the system that everyone works
from — which I'm still building, a year in, because I didn't know I needed
one until I'd lost months without it.

I have some evidence for that beyond the frustration. Two of these builds
had wildly different documentation depth — one had books of design canon,
the other had two markdown files and a handful of briefs. The thin one is
where the drift got worst and where the most work turned out to be built
over something that already existed. That's not a controlled experiment,
and the two builds differed in other ways. But it points the same
direction as everything else: **the protocol catches drift; only the map
prevents it.**

---

## Why I think this might matter to someone else

Most tooling in this space is about **coordination** — keeping agents on
task, managing context, wiring up tools. It assumes the output is
trustworthy and manages the process around it.

This protocol assumes the opposite. It assumes the output is *not*
trustworthy and makes it prove itself.

I don't think a framework can enforce most of what's here, because half of
it is judgment. "Is this text describing a decision or describing a
command?" isn't automatable. "Does this correction contradict something
recorded elsewhere?" isn't either. The mechanical half can be tooled. The
judgment half is where the discipline actually lives.

If you're doing this kind of work and you've found something better, I'd
like to hear about it.

---

*Leslie J. Fleming — chef by trade, 20+ years in kitchens, building
software by directing agents because I can't write it myself. Everything
here was learned the expensive way.*

*The companion piece, on recovering a system you've lost track of, is
at [lost-the-plot](https://github.com/chefsquatch/lost-the-plot).*

Part two — four more rules, with runnable checks: [PART-TWO.md](PART-TWO.md)

Part three — you don't need to write good prompts: [PART-THREE.md](PART-THREE.md)
