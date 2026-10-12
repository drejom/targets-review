---
name: targets-pipeline-standards
description: >
  Conventions for what correct code looks like in an R {targets}-based scientific pipeline:
  report-number traceability, test rigor, pure-function layering, and failing loudly on bad
  input. Read before writing or reviewing any targets/*.R, R/*.R, or .qmd file in a {targets}
  pipeline. Triggers on "write a targets pipeline", "review this R/targets code", "what's
  the convention for a target that produces a report number", "add a test for this pipeline
  function", or any {targets}/Quarto authoring task.
---

# R / `{targets}` / Quarto pipeline standards

You are implementing or reviewing code in a `{targets}`-based R pipeline for a scientific
(or other high-stakes) data analysis. Apply these standards before writing or reviewing any
code. This skill is about **what correct code looks like** — for how review knowledge
accumulates over time, see `review-ratchet`; for how to execute an actual review pass over a
manuscript and its code, see `scientific-review`.

## 1. Report numbers must be derivable, not typed

Every number, table, and figure in a report traces back to a `{targets}` target — never
hand-typed, never computed inline in the report from raw data. Full mechanism, the
`tar_read_raw()` ban, and the file-target re-keying rule: see
`references/report-traceability.md` and `references/qmd-conventions.md`.

## 2. A claim decays — re-verify before trusting it, including your own prior output

Never assume a number, a file's existence, a column's meaning, a naming convention, or a
prior conclusion is still true. Verify against current data/code whenever it matters,
especially before it feeds a claim in a report. This is not restated in full here — see
`adr-practice/templates/principle-claim-decay.md` for the complete principle, mechanisms, and
worked example.

## 3. Tests use synthetic fixtures with a hand-computed known answer, plus dependency injection for I/O

Build a small, explicit fixture where the correct answer is known **by construction** (a
tiny hand-built data frame, synthetic data with a planted known answer), then assert the
function returns exactly that — `expect_equal()` against the known value, not just "doesn't
error."

For anything I/O-driven (reading a file, hitting a service, streaming data), give the
function an injectable source parameter so tests can swap in a fixture and run
deterministically offline, without real files or network access.

Where no prior convention exists for a kind of test, use the standard idiom for that domain
cleanly rather than inventing something ad hoc — you may be setting the precedent other code
follows.

**Tests must earn their keep.** They must never be vacuous, self-evident, tautological,
performative, or ceremonial. Before accepting a test, ask: what edit to the function would
make this test fail? If nothing plausible would, it isn't testing anything.

## 4. General rigor for a `{targets}` pipeline

- **Prefer pure functions in a dedicated source directory** (e.g. `R/`) over inline logic in
  the pipeline script or report chunks — see `references/r-coding-conventions.md` for the
  full layering rule. This keeps everything testable and keeps the dependency graph legible.
- **Fail loudly on malformed, missing, or ambiguous input** — a join that silently produces
  NA, a file that's missing, a parameter that can't be resolved — rather than defaulting or
  coercing silently. Catch problems at the source rather than relying on a downstream
  validation guard to catch what a cleaner upstream check would have caught for free. See
  `adr-practice/templates/principle-trycatch.md` for the specific error-handling version of
  this rule.
- **Don't infer structural facts from naming conventions alone when an authoritative
  metadata source exists** — verify against that source directly. A wrong inference here can
  silently corrupt downstream results without any error. See
  `adr-practice/templates/principle-ssot.md`.
- **Check for prior art before building something that could plausibly already exist** (a
  manifest builder, a parser, a QC check) — but verify any borrowed logic's assumptions
  actually hold for your current data before trusting it; don't copy blindly. See
  `adr-practice/templates/principle-verify-premise.md`.
- **Document *why*, not just *what*,** in comments near any non-obvious decision (a
  threshold, a tolerance, an exclusion, a cutoff) — future readers need the reasoning to
  judge whether the decision still applies as data or scope changes.

## 5. Code layering and naming

Full layering rule, function-prefix conventions, namespace rules, and the DRY-helper check:
see `references/r-coding-conventions.md`.

## 6. Five forbidden patterns

| Forbidden | Why |
|---|---|
| Plotting code in a target's own command body | Untestable without a pipeline run; extract to `plot_*()` |
| Data joins in a report document's code chunks | Hides a computation step from the dependency graph |
| Heavy computation in a report document's code chunks | Should be a pre-built upstream target |
| `tar_read`/`tar_load` inside `R/*.R` functions | Functions stay pure; targets wire data in |
| `source()` inside a target's command | Load helpers once at pipeline startup, not per-target |

## 7. Comments policy

Default: no comments; one line only when the *why* is non-obvious. This matches the general
project-wide comment convention already in force — not restated at length here to avoid a
third copy of the same rule (see `references/r-coding-conventions.md` §6 for the R-specific
version with examples).

## References

| File | Contents |
|---|---|
| `references/targets-semantics.md` | Primary-source `{targets}` package semantics: static dependency analysis, the cross-project file-target bridge, package-function invalidation, dynamic branching |
| `references/r-coding-conventions.md` | Namespace rules, function-prefix conventions, the `R/`/`targets/`/`quarto/` layering rule, DRY-helper checking, comments |
| `references/qmd-conventions.md` | Quarto chapter authoring: setup chunks, the values-object SSoT pattern, palettes, figures-as-file-targets, forbidden patterns |
| `references/report-traceability.md` | The full report-number-traceability mechanism and rationale |
