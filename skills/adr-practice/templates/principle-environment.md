# Principle: the environment that tests is not the environment that runs

"Tests pass" is a claim about the code *plus* the environment it ran in, not a property of
the code alone. A project can easily have four or more meaningfully different environments:

| Environment | What it loads | What it can see |
|---|---|---|
| Local interactive run | source tree, locally installed deps | everything in the working directory |
| CI test job | installed package/build artifact, in a container | may lack optional/system deps, may have a synthetic hostname |
| CI coverage job | the installed artifact specifically | often has **no source tree at all** — paths relative to source resolve to nothing |
| A scheduled/cluster/production job | whatever the runtime environment resolves at execution time | possibly a stale cached copy, a different working directory, no interactive shell |

Each of these has, in practice, produced a failure the others couldn't see. The recurring
mistake is treating a pass as a property of the *code* rather than of the
*code-plus-environment pair*.

## Rules

1. **Name the environment when reporting a pass.** "Tests pass" is an incomplete claim;
   "tests pass locally" and "tests pass in CI" are different, both useful, and not
   interchangeable. When a change touches shared or widely-used code, say which environment
   you verified in rather than reporting a bare pass.
2. **A test that asserts something about the source tree must guard the environment, not
   the fixture.** If a build step runs from an installed artifact with no source tree
   present, a test like "the source file at path X contains Y" needs an explicit guard that
   skips it in that environment — but the assertion must still run unconditionally wherever
   the source tree *is* present. Guarding on environment is not the same as the (banned)
   pattern of guarding on whether the test's own fixture happens to exist.
3. **A dependency that is correctly declared can still be absent in a minimal environment.**
   A missing optional/system dependency in a stripped-down CI image doesn't always *skip*
   affected tests — it can *fail* them, sometimes as a confusing assertion mismatch rather
   than a clean "skip" signal. Guard explicitly on the dependency's actual availability, not
   on an assumption that declaring it in a manifest is enough.
4. **The suite that tests your source is not necessarily the thing that runs in production.**
   If tests call internal functions directly with hand-built arguments, the *actual*
   production entry point (a CLI wrapper, a scheduled job definition, an API route) can drift
   out of sync with the function signatures the tests exercise, and nothing catches it. A
   signature or call-arity check against the real entry point closes this gap more reliably
   than more unit tests of the same internal function.
5. **A job that resolves dependencies at runtime can silently use a stale copy.** If library
   resolution order means a personal/cached copy can shadow a freshly installed one, a
   genuinely-fixed bug can still fail in production while every test is green, because the
   running process never loaded the fix. Pin the resolution path explicitly in any job
   definition that matters.

## Worked example (anonymized)

A test suite passed cleanly both locally and in a continuous-integration "test" job, while a
separate "coverage" job on the same commit reported real failures. The coverage job ran the
suite against the *installed* package artifact, which does not include files that only exist
in the source tree; several tests asserting properties of those source files resolved their
file paths to nothing and failed — not because the underlying assertion was wrong, but
because the environment the coverage job ran in genuinely could not see what the test was
checking. The fix was an explicit "skip only if source tree is absent" guard, not a change
to the assertion itself.

## Consequences

- Treat whichever environment gates merging as *the* gate; treat faster/local environments
  as a useful early signal, not a substitute.
- When a hazard like this is found and fixed, document the fix as a named, reusable guard
  (a helper function, a CI-only skip predicate) rather than a one-off patch, so the next
  occurrence of the same environment gap is caught the same way.
