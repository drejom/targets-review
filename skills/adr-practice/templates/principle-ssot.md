# Principle: every fact has exactly one home

**Independently converged on by two separate projects** with nearly identical phrasing —
treat that convergence as a signal the rule is real, not a house-style quirk.

## The test

> **Could this value have been read instead of computed?**

If yes, computing it instead is a defect — even when the computed answer currently agrees
with the source. "Agreement" and "correctness" are not the same claim.

## Decision

Every fact — a count, a vocabulary, an identifier mapping, a palette, a path — has exactly
one authoritative home. Code that re-derives a fact a table, config, or upstream function
already holds is defective regardless of what it currently returns. When a value must be
read, read it from the one place that owns it; never recompute, restate, or infer it
locally.

## Four properties that let this defect class survive review

1. **Agreement is not correctness, so testing cannot find it.** Several independent
   derivations of the same fact can each individually "mostly work" — agreeing with the
   authoritative source on most cases — while producing three different global answers to a
   simple question ("how many X are there"). Spot-checks and small unit tests pass; the
   defect is architectural, not local. It is found by asking *where a value comes from*,
   never by asking *what it currently equals*.
2. **A diff cannot show it.** `canonical_id = some_other_column` looks locally impeccable in
   a code review — provenance is invisible in the changed lines. Reviewing a diff is
   structurally insufficient for this class; the question has to be asked of the *value*,
   not the patch.
3. **It hides behind a correct-looking local justification.** A one-line fix with an
   accurate comment explaining exactly why it's needed can still be wrong in shape — it
   handles the one case the author saw and silently mishandles the others that share the
   same underlying ambiguity. A good rationale for a local fix is not evidence that a local
   fix was the right shape of fix.
4. **The compensation is the tell.** If a value needs correcting in code at the point of
   use, more than once, across more than one file, in slightly different ways — that is
   evidence the upstream table or source is wrong, not that the correcting code is clever.
   **When you find yourself correcting data in code, fix the data.**

## Worked example (anonymized)

A project needed a "donor" (or equivalent grouping-key) count. Over time, five different
call sites each derived it slightly differently from available columns — one from a raw
identifier string, one from a join key, one from a regex-stripped suffix, and so on. Each
agreed with the authoritative source table roughly 70-85% of the time, which was enough to
pass code review and small tests repeatedly. The five derivations nonetheless produced
*three different totals* for "how many donors does this study have" — a question with
exactly one correct answer. The fix was not a smarter regex; it was a single resolver
function that reads the authoritative table and fails loudly on an unresolvable or
ambiguous identifier, replacing all five derivations.

## Consequences

- Facts are read at a single entry point, never re-derived; identify (and name) that entry
  point for every fact your project treats as canonical.
- **Identity is itself a fact.** A rule phrased around "filters" or "vocabularies" alone can
  miss that an identifier mapping is exactly the same class of defect — state the rule
  broadly enough to cover any derived value, not just obviously "computed" ones.
- **An SSoT is only a source of truth if something checks it.** A table nothing validates is
  just a file — pair the canonical source with a check that keeps it internally consistent
  (no dangling keys, no invented values).
- Prefer a lookup that fails loudly over a transformation that guesses. A function that
  aborts on an ambiguous mapping is strictly better than one that silently returns a
  plausible wrong answer, because the latter is how this defect class stays invisible.
- When a value must be transformed into a genuinely different *representation* of a fact
  (a display label, a composite key built by your own code, a filename derived from an ID),
  that's fine — this rule is about re-deriving a fact recorded elsewhere, not about all
  transformation.
