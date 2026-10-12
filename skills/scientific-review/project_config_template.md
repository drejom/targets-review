# Scientific Review — Project Configuration (optional scratch state)

This file is NOT required to run a review — `SKILL.md` Step 1 discovers these values by
inspecting the target project. Use this template only if you want to write down the
discovered/supplied values for a project you expect to review repeatedly, so a future run
can skip re-discovery. Save your filled copy inside the *target project* (e.g.
`prompts/scientific_review/project_config.md`), not inside this skill — this skill must
stay project-agnostic.

---

## PROJECT IDENTITY

- **Project name:** [e.g. "your-project-name"]
- **Domain:** [e.g. "pancreatic islet single-cell RNA sequencing"]
- **Study type:** [e.g. "observational cohort; no interventions"]
- **Journal target:** [e.g. "Nature Communications" — influences statistical and reporting standards]
- **Stage:** [e.g. "pre-submission draft" / "revision" / "internal report"]
- **Is descriptive only:** [true / false — affects statistical review framing]

---

## FILE PATHS

- **Chapter/manuscript directory (`QMD_DIR`):** [absolute path, e.g. `/home/user/project/quarto/`]
- **Figures directory (`FIGURES_DIR`):** [absolute path, or "none found"]
- **Source-code directory (`R_SOURCE_DIR`):** [absolute path, e.g. `/home/user/project/R/`]
- **SSoT constants file:** [absolute path, or "none found"]
- **SSoT report/values file (`SSOT_FILE`):** [absolute path, or "none found"]
- **Rendered book / HTML (`RENDERED_BOOK`):** [absolute path or URL, or "not available"]
- **Book roots (`BOOK_ROOTS`):** [every Quarto/bookdown project in the repo, plus shared-include directories]
- **Prose style guide (`STYLE_GUIDE`):** [path, or "none" — binding when present]
- **Review standards (`STANDARDS_FILE`):** [e.g. `REVIEW_RATCHET.md` — every reviewer reads it first]
- **Mechanical gates (`GATES`):** [commands, e.g. `Rscript scripts/check_contracts.R` — run before dispatch]
- **Review output directory (`REVIEW_OUT_DIR`):** [absolute path — always supplied by the caller at invocation time, never fixed here]

---

## TARGETS+QUARTO+R LAYER (fill in only if this project uses `targets`)

Leave this whole section blank/absent for a non-targets project — `TARGETS_LAYER` will be
set to "not applicable" and every reviewer/remediator skips it.

- **Targets layer:** [active / not applicable]
- **Pipeline root:** [absolute path — parent of the `_targets/` store]
- **SLURM script:** [relative path, e.g. `logs/tar_make_pipeline.sh`]
- **R helpers:** [relative path, e.g. `R/slurm_helpers.R`]
- **Safe to read in Claude session:** [e.g. reported_values, sample_metadata, qc_summary, qc_thresholds]
- **Never load in Claude session:** [list large targets — Seurat objects, count matrices, H5AD files]
- **Canonical object names:** [the objects that accumulate derived columns as the pipeline progresses]

---

## SSoT PATTERN

Describe how inline numbers are managed in this project, if any such pattern exists:
[e.g. "All inline numbers come from `rv <- targets::tar_read(reported_values)`, built by
`build_reported_values()` in `R/report.R`. Hardcoded numbers anywhere except
`R/constants.R` are SSoT violations. Non-base R functions always use `pkg::fn()` notation
— no `library()` in QMDs."]

If no such pattern exists in this project, write "no SSoT pattern in use" — reviewer 1
will then only check for cross-file numeric disagreement, not SSoT sourcing.

---

## CONTROLLED VOCABULARY

List the preferred term for each concept (reviewer flags deviations):

| Concept | Preferred term | Disallowed alternatives |
|---------|---------------|------------------------|
| [Disease: no diabetes] | [Non-diabetic] | [NODM, ND, non-DM, healthy] |
| [Disease: type 1] | [T1DM] | [T1D, type 1, T1] |
| [Sequencing unit] | library | sample (when referring to a sequencing run) |
| [Biological unit] | donor | subject, patient, individual |
| | | |

---

## KEY CLAIMS TO VERIFY

List the 3-5 most important scientific claims that the Claims & Evidence reviewer
should prioritise for web verification:

1.
2.
3.
4.
5.

---

## STATISTICAL CONTEXT

- **Primary comparisons:**
- **Sample sizes:**
- **Statistical tests used:**
- **Multiple testing:**
- **Note:** [e.g. "All comparative language should be framed as descriptive/observational"]

---

## METHODS REPRODUCIBILITY STANDARD

Target standard for methods completeness:
[e.g. "sufficient for a peer reviewer to assess validity; not required to be fully
reproducible without the raw data. All software versions and key parameters must be stated."]

---

## CHAPTER ORDER (reading order)

1. `index.qmd`
[continue in full reading order — used by R3, R6, R8]

---

## KNOWN ISSUES / OUT OF SCOPE

List anything reviewers should NOT flag (already known, intentional, or out of scope):
-
