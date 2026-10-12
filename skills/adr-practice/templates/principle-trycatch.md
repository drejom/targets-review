# Principle: error handling must not collapse distinguishable failures into one silent outcome

A broad `try`/`catch` (or equivalent — `tryCatch`, `try()`, `purrr::safely()`, a bare
`except:`) that wraps an entire operation and returns the *same* fallback value with the
*same* generic message regardless of what actually failed destroys diagnosability. Worse, in
a pipeline or scheduled-job context, it can make a step that never really ran look
identical to a step that ran and produced a legitimate empty/null result.

## When a catch is appropriate

A catch block is justified only when at least one of these is true:

1. **There are genuinely different recovery paths for different error conditions**, and the
   handler actually branches on which one occurred (by exception type, or by matching a
   specific known message) to take correspondingly different, correct actions — not the
   same fallback regardless of cause.
2. **It exists to re-raise a lower-level error as a clearer, more specific, more actionable
   one.** The point of the catch is to make the failure *more* legible, never less — e.g.
   catching a cryptic low-level traceback and re-raising "dependency X is not installed in
   environment Y — run: `<the fix>`".
3. **The failure is a genuinely per-unit, expected-to-sometimes-fail case** (this one input
   doesn't meet a precondition the operation needs) — and even then, the fallback must
   record a *specific, distinguishable* reason, not a message shared with unrelated failure
   causes.

If none of these apply — if the code is just wrapping "do the real work" and returning an
identical fallback no matter what broke — **use no catch at all** and let the error
propagate with its own message and stack trace. A missing dependency for a core step should
stop the pipeline (or at minimum hard-fail that step), not be silently absorbed into a
result that looks like a valid, completed run.

## Worked example (anonymized)

A pipeline step wrapped an entire external-tool invocation — environment setup, import, and
the actual computation — in one broad catch whose handler returned an all-missing/empty
fallback with a generic warning, regardless of cause. The step reported success on every
run (sub-second, no error) for months. The real cause, eventually found by hand: the
external tool's dependency was never installed in the environment actually being used,
distinct from "the environment itself doesn't exist" and distinct from "ran and legitimately
found nothing" — three different conditions, one indistinguishable fallback. Nothing in the
pipeline's own pass/fail state, metadata, or output revealed which of the three had
happened; a downstream consensus value silently became 100% missing for one of its inputs
without a single error anywhere.

## Consequences

- Prefer **explicit precondition checks** over a defensive catch-all: check for each
  distinguishable failure mode separately (environment exists? dependency available?) and
  fail with a specific, actionable message naming exactly what's missing.
- Reserve any retained catch for the smallest call that can legitimately fail per-unit at
  runtime — not the setup/import steps around it.
- If a soft-fail/skip is genuinely wanted for an optional step, the skip must be recorded as
  first-class, visible output (a boolean flag, a status column) — not just a log line —
  so downstream consumers can tell a true result from a silent fallback.
- A pipeline's success/failure state should reflect whether the real work actually ran, not
  merely that no error was thrown.
