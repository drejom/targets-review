# Quarto (`.qmd`) authoring conventions for `{targets}`-backed reports

Canonical conventions for any `.qmd` (or R Markdown) document that renders from a
`{targets}` pipeline's output. See `references/report-traceability.md` for the full
rationale behind the `tar_read`-only rule below.

## 1. Setup chunk

Every chapter/document has a single `setup` chunk with `#| include: false`:

```r
#| label: setup
#| include: false
targets::tar_load(c(plot_a, plot_b))
values <- targets::tar_read(reported_values)   # the project's single values-object target
source("../R/constants.R")   # only if the chapter needs shared palettes/levels
source("../R/utils.R")       # only if the chapter needs shared helpers
```

Rules:
- No `library()` calls: use `pkg::fn()` in chunks too, as in
  `r-coding-conventions.md` §1. A chapter that attaches packages hides which
  functions it really uses.
- `tar_load()` for plots and larger objects; `tar_read()` for the values object and scalars.
- Do **not** source the same file twice in one chapter's setup chunk.
- Suppress warnings/messages at the YAML level for appendix/validation-only chapters;
  per-chunk `#| warning: false` / `#| message: false` for main narrative chapters.

## 2. Inline values come from one values-object target, by literal name, always

All scalar numbers cited in prose come from the project's single values-object target
(conventionally `reported_values`, read as `values <- targets::tar_read(reported_values)`).
Never hardcode a pipeline-computed number in a `.qmd` file, and never recompute one inline
from raw data in the report. Add new fields to the values-object builder function instead.

```r
`r format(values$total_n, big.mark = ",")`
`r round(values$some_pct, 1)`%
```

**Use `targets::tar_read(name)` with a literal symbol only — never `tar_read_raw()` with a
variable name, and never a hardcoded store path.** `tar_quarto()` (and `tar_render()`)
statically scan a document's source for `tar_read`/`tar_load` calls to infer the document's
dependencies; a dynamic/string-based read is invisible to that scan, so the report can
silently render stale numbers without any error when upstream data changes — see
`references/targets-semantics.md` §1 and §3 for why this defeats static analysis.

### Guard the words, not just the numbers

A value read from the values object can stay correct while the sentence around it goes
stale: "the interval excludes zero", "only in hot-start groups", "two of each". Put a
`stopifnot()` for each such claim in an `include: false` chunk beside the prose, reading the
same values-object fields, so a rerun that changes the data stops the render instead of
publishing a false sentence.

Display helpers may reformat a value (rounding, thousands separators, a typographic minus,
number words); they must not compute one. Arithmetic in an inline expression — a percentage,
a difference, a ratio — belongs in the values-object builder as its own field.

Bare numerals typed into prose and chapter titles are caught mechanically; see §6.

## 3. Palettes and canonical ordering

If the project has a canonical ordering or palette for a categorical vocabulary (cell types,
treatment groups, conditions), source it from `R/constants.R` and apply it consistently —
never ad hoc hex codes or a locally re-typed factor level order in a `.qmd` chunk.

```r
factor(x, levels = canonical_levels)             # before any plot or table using the category
ggplot2::scale_fill_manual(values = group_palette)
```

## 4. Figures as separate `format = "file"` targets

Build each figure as its own `format = "file"` target upstream, and read it with
`tar_read()`/`tar_load()` in the chapter — never build a figure from raw/heavy data inline in
a `.qmd` chunk.

**File targets don't preserve list/vector names after `tar_read()`** — if you read several
file-target paths into one object, re-key them explicitly (e.g. by `basename()` of each
path) immediately after reading, rather than assuming name-preservation you haven't verified.

Every figure chunk needs:
```r
#| label: fig-<kebab-case-id>
#| fig-cap: "Caption text."
#| fig-width: <n>
#| fig-height: <n>
```
- Pre-rendered image files: `knitr::include_graphics(path)` with the same chunk options,
  omitting `fig-width`/`fig-height`.
- Tables use the `tbl-` prefix and `#| tbl-cap:` instead of `fig-cap:`.

### Rendered size

A figure saved at 9 in and 200 dpi is 1,800 px wide; in a 750 px text column the browser
scales it to about 40%, and 9 pt axis text renders near 8 px. Check figures at their
rendered size, not in the PNG viewer: measure the rendered width (a headless browser
reading `getBoundingClientRect()` works) and aim for text of at least 11 px. Widen
multi-panel figures with `column: page` or `column: body-outset` rather than shrinking text.
Hidden tab panels report a width of zero; activate them before measuring.

## 5. Heavy objects are forbidden in `.qmd` files

Never load a large in-memory object (a full dataset, a trained model, a large matrix)
directly in a report document. All plots and tables needing heavy objects are pre-built as
pipeline targets; the `.qmd` only calls `tar_load()`/`tar_read()` on the already-built
result.

## 6. Guard the prose mechanically

Convention alone does not keep typed numbers out; a test does. Add one that
strips code chunks, YAML and inline `` `r ...` `` values from every `.qmd`, then
fails on any remaining digit outside an explicit allowlist (scale definitions
such as "0, typical testis", identifiers such as clutch or batch codes, figure
and appendix numbers, `log~2~`).

```r
prose_numbers <- function(path) {
    text <- paste(readLines(path, warn = FALSE), collapse = "\n")
    text <- gsub("(?s)```.*?```", "", text, perl = TRUE)
    text <- gsub("`r [^`]*`", "", text)
    # ... strip links, citations and allowlisted patterns, then:
    regmatches(text, gregexpr("(?<![A-Za-z0-9.])[0-9]+(\\.[0-9]+)?", text, perl = TRUE))[[1]]
}
```

Give the guard its own fixture test (a literal is caught, an inline value is
not), or it can pass vacuously. Incident: a report audited as clean still had a
typed "95%" interval, a typed "36 °C" in Methods and a percentage computed
inside a chapter; the guard found all three.

Three places the guard cannot reach need their own check:

- **Headings.** Inline `r` in a chapter title renders blank in Quarto's sidebar
  and TOC. Keep the title literal and assert it in the setup chunk, e.g.
  `stopifnot(values$design$window_end == 9)`, so a design change stops the
  build instead of leaving a false title.
- **Labels typed into figure code** ("28 °C", stage vectors, reference-line
  text). Check them against the values object in one function called when the
  figures are built.
- **Plot annotations that restate a reported number** (n labels, group means,
  ratio labels). Read them from the values object, or assert they equal it; a
  second computation of the same count agrees today and drifts later.

## 7. Mirrors are not sources

A copy of the report published elsewhere (an HTML page, slides, an email
summary) is outside the guard. Generate it from the exported values, or
reconcile every number in it against a dump of the values object before it is
shared. Incident: a hand-built web mirror of a book was reconciled number by
number against `reported_values` before release; nothing else would have
caught a stale value there.

## 8. What NOT to do

| Forbidden | Use instead |
|-----------|-------------|
| Loading a heavy object directly in a `.qmd` | Pre-build as a pipeline target |
| Hardcoded scratch/store paths | A named path constant from `R/constants.R` |
| Ad hoc hex codes for palettes | Named palettes from `R/constants.R` |
| Reimplementing a shared plot/table helper inline | The shared helper from `R/utils.R` |
| Bare function calls without `pkg::fn()` | Always `::` notation, matching `r-coding-conventions.md` |
| `source()`-ing the same file twice in one chapter | Once per setup chunk only |
| Inline recomputation of a values-object field | Add the field to the values-object builder |
| `tar_read_raw()` with a variable target name | `tar_read(literal_name)` only |
| `library()` in a setup chunk | `pkg::fn()` in every chunk |
| Inline `r` in a heading | A literal heading guarded by `stopifnot()` |

## 9. Report independence

The report should be renderable independently from the pipeline's exported artifacts,
without requiring a fresh analysis run — see `references/report-traceability.md`.
