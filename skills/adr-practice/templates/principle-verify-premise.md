# Principle: verify the premise before implementing

Work done correctly against a false premise is not safe work — it produces a confident
wrong answer that survives review precisely because the execution was competent. The cost
is not wasted effort; it is published wrong conclusions and deleted correct work, because a
plausible premise plus careful execution looks exactly like a correct result.

False premises tend to come from sources that *feel* authoritative: a prior diagnostic run,
a code comment, an issue's own recommendation, a cross-check's silence. None of these is
evidence about the *current* state of the system.

## Four recurring sources of false premises, and the check for each

### 1. A diagnostic is evidence only about the code that was running when it ran

Check its date against every change since — **especially your own.** Re-running a stale
diagnostic is cheap; citing it without re-running it is not. A result that was true before a
dependency, a parameter, or an upstream input changed is not evidence about the current
behavior.

### 2. A written justification is not evidence

Not a code comment, not an issue's own stated rationale, not a decision doc marked
"Accepted." A justification tells you what someone believed when they wrote it — that is a
lead to investigate, not a finding to act on. An exclusion, a workaround, or a special case
can persist for a long time with a plausible-sounding reason attached, entirely independent
of whether the reason is still (or was ever) correct.

### 3. Check a cross-check's vocabulary before reading its disagreement as evidence

Silence, or disagreement, from a method that *cannot express* the answer you're looking for
is not a second opinion. Before treating "method B disagrees with method A" as informative,
confirm method B's vocabulary can actually represent the claim in question — a mismatch in
expressiveness looks identical to a genuine disagreement, but means nothing.

### 4. A standard documented elsewhere in the repo is not applied by that fact alone

A statistical or architectural standard can be correctly written down in one place and
silently not followed at a specific call site in another file. Grep for the standard at the
*site in question*, not just in the repository's documentation — a standard's existence
proves nothing about its application.

## Worked example (anonymized, composite)

A diagnostic run showed a particular code path recovering an expected signal. Later changes
to an upstream configuration were shipped in the same project, and nobody re-ran the
diagnostic before citing it again to justify deleting a caveat from a published document.
With the current configuration, the code path no longer recovered that signal at all — the
diagnostic had become stale the moment the upstream change shipped, and the caveat it was
used to justify removing was the correct one.

## Consequences

- A review finding of the shape "this premise was not checked" is first-class, independent
  of whether the conclusion it supported happened to be right — correctness by luck does not
  excuse skipping verification.
- Where a premise genuinely cannot be checked cheaply, state it explicitly as an assumption
  in the work rather than proceeding silently as if it were established.
- This is the review-time counterpart to "fail loudly on malformed input" — one applies
  continuously during implementation, the other applies once before starting.

## Related

- `principle-ssot.md` — a false premise about *where a value comes from* is a special case of this
- `principle-claim-decay.md` — the companion principle for *written* claims specifically, including this very template
