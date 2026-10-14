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
- Use `pkg::fn()` in chunks as in `R/` (no `library()` calls); this keeps §6's namespace rule
  true for reports too.
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

A test that scans `.qmd` prose (outside code and inline expressions) for bare numerals
catches hand-typed values cheaply. Allow-list the few legitimate literals: scale anchors
("0 = testis"), notation (log~2~), identifiers and years.

### Chapter titles

Do not put inline R in a chapter title: Quarto builds the sidebar and page titles before
the chapter executes, so navigation shows a blank where the value should be. Keep the title
literal and guard the literal with a `stopifnot()` in a setup chunk placed above the title.

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

## 6. What NOT to do

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

## 7. Report independence

The report should be renderable independently from the pipeline's exported artifacts,
without requiring a fresh analysis run — see `references/report-traceability.md`.
