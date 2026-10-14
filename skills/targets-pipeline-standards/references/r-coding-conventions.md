# R coding conventions for `{targets}` pipelines

Canonical standards for `R/*.R` and `targets/*.R` code in an R `{targets}`-based pipeline.
Adapt the specific examples to your project's own domain; keep the structure.

## 1. Namespace — always use `::`

Never call `library()` inside sourced `R/` files or `targets/*.R` files. Use `pkg::fn()`
everywhere in pipeline/package code.

```r
# GOOD
dplyr::filter(df, x > 0)

# BAD
library(dplyr); filter(df, x > 0)
```

`library()` is permitted only in interactive scratch scripts and test files — never in
pipeline source.

This also matters for `{targets}`'s static dependency analysis: a namespaced call
(`pkg::fn()`) is *not* registered as a dependency on `fn()` by `targets` itself — see
`references/targets-semantics.md` §1. The namespacing convention here is about code
hygiene; be aware it has a separate, real effect on the dependency graph too.

## 2. Prefer the ecosystem's higher-level tools over base-language loops where the project already has one

If your project has standardized on a higher-level data-manipulation toolkit (tidyverse-style
verbs, a specific tensor/array library, etc.), prefer it consistently over hand-rolled
base-language loops for the same operation — not because loops are wrong, but because mixing
idioms for the same operation makes the codebase harder to read as one system.

```r
# GOOD (tidyverse-standardized project)
purrr::map(list, fn) |> dplyr::bind_rows()

# Inconsistent with a tidyverse-standardized project
lapply(list, fn); do.call(rbind, lapply(...))
```

## 3. Function naming conventions

Pick a small set of verb prefixes and apply them consistently; a reader should be able to
guess a function's shape (pure vs. I/O, cheap vs. heavy) from its name alone. A workable
starting set for a `{targets}` pipeline:

| Prefix | Purpose | Typical location |
|--------|---------|------|
| `build_*` | Constructs data objects; may read files or do I/O | `R/*.R` |
| `plot_*` | Pure plotting function — takes data, returns a plot object; **no I/O, no joins** | `R/plots_*.R` |
| `compute_*` | Heavy transforms (dimensionality reduction, statistical models, large aggregations) | `R/*.R` |
| `run_*` | Submits external jobs or processes (cluster jobs, subprocess calls) | `R/*.R` |
| `get_*` | Retrieves or looks up a value | `R/utils.R` |
| `decode_*` | Maps encoded values to human-readable labels | `R/utils.R` |
| `filter_*` | A canonical, reusable row filter for a shared dataset | `R/utils.R` |

## 4. Layering rules — where code lives

```
R/constants.R       ← magic numbers, enumerated vocabularies, palettes, excluded IDs
R/utils.R           ← shared helpers used across the codebase
R/plots_*.R         ← plot_* and compute_* functions; no targets, no tar_read
R/report.R          ← the report's single values-object builder (see report-traceability.md)
targets/*.R         ← target definitions only; function bodies live in R/
quarto/*.qmd        ← tar_read/tar_load, inline values from the values object, plotting calls
                       — no joins, no heavy computation
```

### Forbidden patterns

- **No plotting code inside a target's own command body** — extract to a `plot_*()` function.
- **No data joins inside a report document's code chunks** — join in a `build_*` function or
  a dedicated target.
- **No heavy computation inside a report document's code chunks** — use a `compute_*` target
  pre-built upstream.
- **No `tar_read`/`tar_load` inside `R/*.R` functions** — functions stay pure; targets wire
  data into them. Mixing the two makes functions untestable without a running pipeline store.
- **No `source()` inside a target's command** — load all helpers once at pipeline startup.

## 5. Single source of truth

### Constants

All magic numbers and shared vocabularies live in one constants file. Never hardcode a
value, a threshold, or a named vocabulary element anywhere else:

```r
EXCLUDED_IDS       <- c("...")
THRESHOLD_FACTOR   <- 3L
group_palette      <- c("GroupA" = "#4393C3", "GroupB" = "#D6604D")
canonical_levels    <- c("TypeA", "TypeB", "...")   # canonical ordering
```

### Metadata tables — read-only, derived from

If your project has authoritative metadata tables (sample sheets, library inventories,
lookup tables), treat them as read-only inputs that everything else derives from. Never
modify them in-pipeline, and never re-derive a value they already hold — see
`adr-practice/templates/principle-ssot.md`.

**Derive groupings from the canonical design table, never from a hand-typed mapping.** A
plot or summary that re-types which groups belong to which condition silently drops or
misassigns rows when the typed list and the table disagree. Join to the design table and
stop on any unmatched row. Incident: a figure's hand-typed group list omitted one control
group; three embryos vanished from the figure with no error.

**Apply corrections once, to every copy.** When a sample is relabelled or excluded in
configuration, every dataset that contains that sample — including secondary copies built
for another analysis — must take the correction from the same configured list, and a check
must stop the build if any copy disagrees. Incident: a genotype relabel lived in one
profile's config while a second copy of the same dataset, read straight from the registry,
kept the old label.

### Two silent R pitfalls

- **Data masking.** Inside `dplyr` verbs a column shadows an environment variable of the
  same name: with a column `p`, `filter(.data$fdr < p$cutoff)` reads the column. Use
  `.env$p` (or `!!`) for environment values whenever a column could share the name.
- **Names travel.** Named numeric vectors (e.g. scores named by sample ID) carry their names
  into `unlist()`ed values objects and exported tables. `unname()` before a value leaves
  the pipeline, or identifiers leak into reports.

### DRY enforcement — canonical helpers

Before writing inline logic that looks like it might recur, check whether a shared helper
already exists for the pattern. Keep a running table in your project's own style guide, e.g.:

| Pattern | Use instead |
|---------|------------|
| `group_by + count + mutate(prop = n/sum(n))` | a shared `add_group_pct()`-style helper |
| A repeated theme/styling block | a shared `theme_*()` helper |
| A repeated ID-normalization transform | a shared `normalise_*_key()` helper |
| A repeated domain-specific filter | a shared `filter_*_canonical()` helper |

If a helper doesn't exist yet, add it to the appropriate `R/` file before using the same
logic in two or more places — not before, since a single use isn't yet a pattern.

## 6. Comments

Default: no comments. Add one only when the **why** is non-obvious — a hidden constraint, a
workaround for a specific bug, a threshold choice that needs justifying. This mirrors the
general project-wide comment policy already in effect for this plugin's own authoring; it's
not restated at length here — see your global agent instructions for the full rule.

```r
# GOOD
gsub("NO$", "", key)   # strip a known-bad trailing suffix from legacy IDs

# BAD
dplyr::filter(df, n > 0)   # filter rows where n is greater than zero
```

Never write multi-line comment blocks or docstrings in pipeline source. One short line max.
