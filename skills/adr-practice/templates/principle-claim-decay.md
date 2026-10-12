# Principle: a claim decays from the moment it is written, and only re-verification resets it

Never assume a number, a file's existence, a column's meaning, a naming convention, or a
prior conclusion — yours or someone else's — is still true. Verify against the actual
current data or code whenever it matters, especially before it feeds a published claim.
Stale documentation, stale design notes, and stale "this worked before" are a defect class,
not a convenience to lean on.

## Why this class is uniquely dangerous

A wrong claim in prose does not fail. Tests pass, the pipeline runs, the figure renders. It
fails only when a human **acts** on it — and by then the claim has the authority of having
survived unchallenged. It is self-reinforcing: the more central a claim is, the more places
cite it, and each citation looks like corroboration while adding no actual verification.

The underlying shape is always the same: **something true was written down, the world
moved, and the writing did not.** The defect is not that the claim was wrong when made (it
may have been perfectly true then) — it is that nothing re-checked it, and nothing made its
age visible. No document is exempt, including the one you are reading now, and including any
review-standards or ADR document whose entire job is preventing this.

## Three mechanisms, in order of strength

### 1. Derive it — the claim cannot drift because it is not stored

Assemble a published value from its source via a function or query, and test that the
function agrees with the underlying computation. A derived claim has no age. Prefer this
whenever the cost is justified by how often the claim is read or how consequential it is if
wrong.

### 2. Date it — the claim is evidence, not a standing property

A measured number stated in present tense asserts something it cannot actually support once
time has passed. Split by surface:

| Surface | Rule |
|---|---|
| User-facing prose, published docs, in-code docstrings | carry the *reason*, and cite where the supporting figure/measurement lives, rather than restating the number as current fact |
| Internal records — decision docs, this kind of checklist, code comments, issue/ticket bodies | keep figures **as dated evidence** ("measured 2026-08-24: ...") |

The split is the point: published surfaces outlive their measurements; dated internal
records are read as history, not as a standing claim.

### 3. Re-verify before acting — the only defence against an inherited claim

Before implementing against any claim — a decision doc, an issue title, a comment, a figure
caption, a number in a report — check it against the current code or data. If it is wrong,
correct it **in the same change**, and say so. An issue or ticket that has been open a long
time is evidence about the *past*, not the present.

## Worked example (anonymized)

A written vignette/report stated a specific quantitative pattern that was true when
measured ("no case had all three methods agree, and wherever two of three did, the winning
method was always the same one"). Both halves were true at the time and both had since
become false — a later refinement of the underlying method had moved the numbers
substantially. In the same project, a bug report was filed citing that stale vignette
number as if it were current, and the report turned out to be invalid: the reporter had read
the document instead of re-measuring. Separately, two older issues were closed on a
five-minute re-measurement, because the defects they described had already been fixed
months earlier by unrelated work — nobody had closed them because nobody had re-checked.

## Consequences

- "Is this still true?" is a routine question to ask of any claim you're about to build on,
  not an accusation of whoever wrote it.
- Correcting a stale claim is expected maintenance — framing it that way makes it cheap to
  do and cheap to admit, rather than something to quietly route around.
- Automating staleness detection is mostly infeasible (no check can know that a narrative
  claim has stopped being true) — this is why it lives in judgement rather than tooling,
  with one partial exception: a decision doc's own `Status` field (Proposed/Accepted/
  Deprecated/Superseded) *can* be checked mechanically for staleness signals, even if its
  *content* can't be.

## Related

- `principle-verify-premise.md` — the implementation-time counterpart: verify before you
  start building, not just before you cite a result afterward.
