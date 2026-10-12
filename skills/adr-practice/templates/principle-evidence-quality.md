# Principle: count sources, not checks — and a failing control invalidates the method

Three techniques applied against one underlying system are **one check**, not three.
Triangulation requires a genuinely different system with a genuinely different failure mode
— not three ways of asking the same database the same question. Never write "verified N
independent ways" without naming N *systems*, because the sense of corroboration from
running multiple techniques against a single source is false confidence, not evidence.

## Rule 1 — a control that disagrees with your method invalidates the method

If you have a known-good case (a control) and your method gives the wrong answer for it,
**stop and fix the instrument before trusting any of its other answers.** A failing control
is never a curiosity to note in passing and continue past — it means the method itself is
broken for *all* of its outputs, not just the control case. Continuing to use a method after
observing it fail its own control, on the reasoning that the control case is "known to be
difficult" or similar, converts a falsified method into an unfalsifiable one.

## Rule 2 — count systems, not techniques

Range-fetching bytes from a convenience API, reading a metadata field from the same API, and
checking a second endpoint of the same API are **one source checked three ways**, not three
sources. A genuine second source is a different system with an independent failure mode —
e.g. the authoritative origin system rather than a distribution layer's cache of it. Before
writing "confirmed by N methods," ask whether any two of those methods could plausibly share
a root cause for being wrong; if so, they are one method.

## Rule 3 — a finding that contradicts a written claim needs better evidence than the claim it overturns, not just newer evidence

When a new finding disagrees with an existing, previously-reasoned decision or note, the
default prior is that the existing claim won its argument already — it was presumably
written by someone reasoning carefully at the time. Reconcile the disagreement before
overwriting the prior claim. Being unable to explain *why* the prior author believed the
opposite is itself the signal to stop and investigate further, not a reason to proceed. This
is the mirror image of claim decay (`principle-claim-decay.md`): that principle says a claim
decays until re-verified; this one says the re-verification itself must be *better
evidenced* than what it overturns, not merely more recent.

## Worked example (anonymized)

An investigation concluded that several external datasets were unusable because a
convenience data-distribution layer reported no retrievable content for them. The same
layer *also* reported nothing for a known-good control dataset that was independently known
to work — observed, noted as "this layer isn't reliable for this case," and then used
anyway to reach the same negative conclusion for the other datasets. Three different
techniques against that one convenience layer (byte-range fetch, one metadata field, a
second metadata field) were written up as "verified three independent ways" — one source,
three techniques. A second, genuinely independent source (the origin system's own
accounting) showed the data had been present the whole time. A correct prior note explaining
exactly why the data should be recoverable was overwritten with the new, wrong conclusion,
rather than being reconciled with it first.

## Consequences

- When reporting confidence in a finding, name the systems checked, not the number of
  queries run.
- Treat a failing control as a stop condition for the *method*, not a footnote on the result.
- Before overwriting a previously-reasoned document with a new conclusion that disagrees
  with it, explain — to yourself, in writing — why the earlier author was wrong. If you
  can't, that's the signal you're missing something, not that they were.
