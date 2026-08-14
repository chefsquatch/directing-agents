# You Don't Need to Write Good Prompts

**What actually happens when I direct a coding agent, and where the skill
turned out to be.**

Leslie J. Fleming · 2026

Third in a series with
[part one](https://github.com/chefsquatch/directing-agents) and
[part two](https://github.com/chefsquatch/directing-agents/blob/main/PART-TWO.md).

---

## The thing everybody gets wrong about this

There's a whole cottage industry telling you to learn prompt engineering.
Write clearly. Be specific. Use examples. Structure your request.

I don't do any of that, and I've been directing agents on a real codebase
for a year.

Here's an actual thing I typed:

> `also need to train the render scout model with all of this work in
> kernle right? that was the idea train both same time if we need the
> renderer later its ready to go?`

Typos, no capitals, no punctuation to speak of, a question mark doing
work three clauses can't support. What came back was a three-page
specification with the constraints named, the failure modes fenced, the
verification listed, and the parts I hadn't thought about surfaced as
questions for me to answer.

That spec is what the coding agent executed against. Not my sentence.

---

## The two-layer thing

Once I saw it, I couldn't unsee it. There are two different jobs:

**Layer one — intent to specification.** I say what I want, badly. A
model turns it into the detailed, structured, constraint-laden document
the tooling actually needs. This is translation, and models are
extremely good at it.

**Layer two — execution.** A coding agent takes the specification and
writes the code.

Prompt engineering advice is aimed at people trying to do layer one
themselves. That's the part you can hand off. You don't need to learn to
write a three-page brief. You need to say what you mean and let something
else format it.

That sounds like I'm claiming the skill floor is zero. I'm not — I'm
claiming it moved.

---

## Where it moved to

**You have to be able to reject a good-looking spec.**

This is the whole thing. The translation layer produces documents that
are fluent, structured, confident, and sometimes wrong. Not wrong in a
way that looks wrong. Wrong in a way that reads like exactly what you
asked for.

Things I've had to reject from specs written for me, all of which looked
fine:

- A file path that didn't exist. The spec cited it three times.
- An instruction built on a rule that had been superseded a week earlier.
- A "load-bearing" clause that I asserted myself, from checking one case,
  which turned out to be false when the agent enumerated all of them.
- A symbol named confidently throughout — which had been invented by
  mistake earlier that same day and never existed in the code.

Every one of those would have produced a session that did the wrong thing
efficiently, reported success accurately, and cost a week.

**The scarce skill is not articulation. It's verification.** Knowing what
you want well enough to catch a plausible document being wrong about it.

---

## Why this is good news if you can't code

I can't write code. What I can do is hold an architecture in my head and
tell when a description of it is off.

That turns out to be the half that doesn't delegate. A model will
translate my intent into a spec all day. It won't notice that the spec
contradicts a decision I made in March, because it wasn't there in March
and it has no way to know what I actually said.

There's a measured version of this. An agent I direct kept a
hand-classified record of its own errors, naming who caught each one, and
the split is clean:

| error type | who caught it |
|---|---|
| a single retrievable value — a count, a duration, "exactly N" | **itself**, in seconds |
| a frame, an attribution, a joint between two true facts | **the human**, or nobody |

The value errors don't reach you. What reaches you is a document where
every individual claim is true and the sentence connecting them is
invented, or where a rule is attributed to you that you never said.

So the useful question to ask about a spec isn't *is this correct.* It's:

1. **Does this joint exist anywhere, or did the connection get made up?**
2. **Did I actually say this, or did something compose it and attribute
   it to me?**

Neither of those requires reading code. Both of them require knowing your
own system better than anything holding it for eight minutes at a time.

---

## What I actually do

The whole loop, in practice:

**I say the thing badly.** Whatever's in my head, in whatever shape it
arrives. Half-sentences, wrong terms, the idea before it's finished. I
don't clean it up — cleaning it up is the translation layer's job and
it's better at it than I am.

**Something turns it into a spec.** Constraints named, verification
stated, the things I didn't think about raised as questions rather than
assumed.

**I rule on the spec.** This is the actual work. Not "is this well
written" — it always is. *Is this what I meant, does it contradict
anything I've decided, is anything in here invented.*

**The coding agent executes the spec.** Not my sentence. The spec.

**I check the report against the spec, not against the code.** Which is
the only kind of checking I'm capable of, and it turns out to be enough
if the spec was specific about what proof looks like.

---

## The honest limits

**This fails badly if you don't know your own system.** Everything above
rests on being able to tell when a fluent document is wrong. If you can't
— if the spec is the first time you're thinking about the design — you're
not directing, you're being led, and the fluency is actively working
against you.

**It fails if you accept the first draft.** The specs I get back are
wrong often enough that accepting them uncritically would be worse than
writing bad ones myself. A clumsy spec I wrote fails visibly. A fluent
spec that's wrong fails weeks later in a place I won't connect to it.

**And I can't verify the code.** That's a real gap and I'm not pretending
otherwise. I compensate with a protocol that makes agents prove things
rather than assert them, which is what the other two documents in this
series are about. It's a substitute, not an equivalent.

---

## The short version

Learning to write great prompts is optimizing a layer you can hand off.

What you can't hand off is knowing what you want precisely enough to
catch a well-written document being wrong about it — because that's the
one thing nothing else in the loop has access to.

Say it badly. Let something else make it precise. Then be the person who
can tell when precise isn't right.

---

*Leslie J. Fleming — chef by trade, 20+ years in kitchens, building
software by directing agents because I can't write it myself.*
