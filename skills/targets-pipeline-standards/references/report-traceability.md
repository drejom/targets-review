# Report traceability: every number in the report must be derivable, not typed

All substantial computation — statistical summaries, tables, and plot generation — belongs
in the `{targets}` pipeline, not in the report document. The report should be independently
renderable from exported artifacts, without requiring a fresh analysis run. If your report
is a Quarto book: use the "cosmo" theme by default, and use `downloadthis` (or an equivalent)
when artifact download needs to be offered to the reader.

## The rule

Every number, table, and figure cited in a report's narrative must trace back to a
`{targets}` target — never hand-typed, never computed inline in the report document from raw
data.

## The mechanism

Use a single values-object-building target (conventionally named something like
`build_reported_values()`, materialized as a target called `reported_values`) that returns
one named list — **the single source of truth for every inline narrative number.** Validate
it with a guard function that hard-stops the pipeline build if any value is `NULL`, empty,
`NA`, or non-finite. A blank or `"NULL"` string reaching a published report is a **build
failure**, not a rendering detail to patch around afterward.

Every report document reads this list via a plain, literal `tar_read(reported_values)` call
— see `qmd-conventions.md` §2 for the full authoring convention.

## Why `tar_read_raw()` (or any dynamic/string-based read) is banned for anything a report depends on

`{targets}` builds its dependency graph via **static code analysis** — it scans source for
literal symbol references like `tar_read(some_name)`. A string-based or dynamic read (e.g.
`tar_read_raw(variable_holding_a_name)`) is invisible to that scan. This means:

- the report silently **stops tracking what it actually depends on**;
- the pipeline's dependency graph no longer reflects reality;
- the report can render **stale numbers without any error**, because `{targets}` has no way
  to know it needs to invalidate and re-render.

Target references in a report must be literal symbols, full stop — see
`targets-semantics.md` §1 and §3 for the primary-source basis of this rule.

## Figures as file targets, re-keyed after reading

Build figures as separate `format = "file"` targets (see `targets-semantics.md` §4 for the
format's full semantics). Because file targets **don't preserve list/vector names** after
`tar_read()`, re-key them — e.g. by `basename()` of the file path — immediately after
reading, rather than relying on name-preservation you haven't actually verified for your
`{targets}` version.

## Display fits are not evidence

A model fitted only to draw a figure (a pooled reference curve, a smoother, a
landscape across every group) gets its own target, is labelled as a display
fit in its caption, and its statistics stay out of the values object and the
prose. Statistics of a pooled fit often measure sampling composition, not the
effect: incident, a density landscape pooled across genotypes had a deep
"ovary" well guaranteed by the genotype that always makes ovaries, and its
barrier heights tracked how many of each group were sequenced.

## Names on exported vectors

`unname()` numeric vectors before they enter the values object or an export.
Names carried from a per-sample table can put sample identifiers into a
report or a shared bundle without anyone typing them.

## Related

- `qmd-conventions.md` — the full report-authoring convention this rule feeds into.
- `adr-practice/templates/principle-ssot.md` — this rule is the `{targets}`-specific
  instance of "every fact has exactly one home": the values-object target *is* the one home
  for every number a report cites.
- `adr-practice/templates/principle-claim-decay.md` — a claim in the report decays exactly
  like any other claim; deriving it (this mechanism) is the strongest of the three defenses
  that template describes, which is exactly why this rule is mandatory rather than advisory.
