---
name: scientific-review
description: >
  Run a multi-agent scientific review of a manuscript, analysis book, or report (Quarto,
  R Markdown, LaTeX, or plain markdown chapters) and its supporting analysis code, on ANY
  project. Dispatches 10 independent reviewer roles with non-overlapping lanes (accuracy &
  provenance, claims & evidence, reader experience, statistical design & analysis
  implementation, methods reproducibility, internal consistency, visual design,
  terminology, taxonomy integrity, constant integrity), verifies their findings against the
  code and data, then on request runs a remediation pass. Also handles collaborator-feedback
  triage with a dual-audience (technical + plain-language) response. On a targets+Quarto+R
  project it adds that pipeline's contracts automatically. Use whenever asked to "review
  this book/manuscript/report", "run a scientific review", "check this paper for errors
  before submission", "audit these chapters for accuracy/consistency/overclaiming", or
  "triage this feedback". Discovers project structure rather than requiring a config.
---

# Scientific Review

Announce at start: "I'm using the scientific-review skill."

A whole-corpus review of reported science: prose, captions, rendered pages **and the code
that produces the numbers**. It is not a diff review. The defects that matter most are
properties of the whole codebase, invisible in any change set. Examples:
- a normalisation computed over the wrong feature set;
- a count taken over one population and divided by another;
- a "Harlan" table that silently includes another cohort;
- a direction word left standing after a rerun reversed it.

The prose around such a number is usually fluent and self-consistent. It is expensive
because it must be complete; do not narrow it to recent changes.

Reviewers are independent subagents. Each reads the sources fresh, so no role anchors on
another's conclusions. Each role owns a lane (`reviewers/00_common.md` §5), so a problem is
reported once. **Reviewers are evidence, not oracles.** Every finding is verified by the
orchestrator before it reaches the user (REVIEW Step 5).

---

## Step 1 — Discover the project

Do not ask the user to fill in a config file. Discover by inspection; ask only for what is
genuinely unguessable, and otherwise proceed on stated defaults.

| Value | How to discover it |
|---|---|
| `BOOK_ROOTS` | Every directory with a `_quarto.yaml`/`_quarto.yml`/`_bookdown.yml` (a repository may hold several books), plus any directory of shared include files they reference. |
| `CHAPTER_ORDER` | Per book root, the `chapters:`/`rmd_files:` list. Files on disk but not in the list are out of the book. Note them; do not review them. |
| `RENDERED_BOOK` | Per book root, the output directory (`_book/`, `_site/`, `docs/`) if it holds an `index.html`. Note its modification time against the sources: a stale render must be rebuilt or flagged. |
| `FIGURES_DIR` | Directories holding files that figure chunks reference. |
| `R_SOURCE_DIR`, `PIPELINE_DEFS` | Analysis code (`R/`, `src/`, `scripts/`); pipeline definitions (`_targets.R`, `targets/*.R`). |
| `SSOT_FILE`/`SSOT_PATTERN`, `CONSTANTS_FILE` | The canonical values object and constants module, described in one sentence each, or "none". |
| `STYLE_GUIDE` | A prose/authoring guide (`spec/*prose*`, `spec/*style*`, `STYLE.md`, a style section in `CLAUDE.md`/`AGENTS.md`). Binding when present. |
| `STANDARDS_FILE` | The project's accumulated review knowledge (`REVIEW_RATCHET.md`, `.claude/agents/docs/review-standards.md`, or a standards section referenced from `CLAUDE.md`). Every reviewer reads it first. |
| `GATES` | Mechanical checks (`scripts/check_*.R`, lint/test commands named in `CLAUDE.md` or CI). |
| `CONTROLLED_VOCAB`, `KNOWN_ISSUES` | From the style guide, standards file or `CLAUDE.md` if present. Ask the user for known issues if none are written down. |
| `DOMAIN`, `STUDY_TYPE`, `JOURNAL_TARGET`, `KEY_CLAIMS` | Infer from the manuscript (key claims are the chapter Summary claims). Name the defaults in the report. |
| `REVIEW_OUT_DIR` | Caller-specified. Default `<repo>/.review_output/<date>/`. Never a path inside this skill. |

`project_config_template.md` records these for a repeat review. It is optional.

### Step 1b — Targets+Quarto+R layer

If `_targets.R` or `targets/*.R` exists, set `TARGETS_LAYER: active` and save
`targets::tar_manifest(fields = c("name", "command"))` to `TARGETS_MANIFEST`. Otherwise set
it to `not applicable`. Every reviewer skips `[TARGETS LAYER]` sections; nothing fails for
lack of them.

**Safety:**
- Safe to read in session: `tar_meta()`, `tar_manifest()`, the values object, and small
  tables.
- Never load in session: single-cell objects, count matrices, or any target over ~50 MB.
  Checks on those go to the cluster, or are marked UNVERIFIED with the exact check needed.

---

## Modes

| Invocation | Mode |
|---|---|
| `/scientific-review` | **REVIEW**: 10 reviewers, verification, report |
| `/scientific-review remediate` | **REMEDIATE**: apply verified mechanical fixes and approved decisions |
| `/scientific-review feedback [text or file]` | **FEEDBACK**: collaborator feedback triage |

---

## MODE: REVIEW

### Step 2 — Pre-flight

1. **Rendered output is current.** If sources are newer than `RENDERED_BOOK`, rebuild first
   or tell the user; reviewing a stale render wastes the rendered-page checks.
2. **Run the gates** (`GATES`) and capture their output. Reviewers must not re-report what a
   gate already checks (`00_common.md` §4); a failing gate is reported once, here.
3. **Snapshot** (targets layer): `tar_meta(fields = c("name", "time", "error"))`. Errored
   or never-built targets are findings in their own right.

### Step 3 — Dispatch 10 reviewers in parallel

For each reviewer:
- Read `reviewers/00_common.md` and the role file.
- Build its prompt: config values, gate output summary, `KNOWN_ISSUES`, then the two files'
  contents or paths, with the instruction to read `00_common.md` first.
- Dispatch all 10 at once with `run_in_background: true`.

Use the strongest available model for R1, R2, R4 and R5. Their findings turn on judgement
about code and evidence, and a weak finding there costs more to verify than it saves.

| Agent | File | Lane | Web |
|---|---|---|---|
| R1 | `01_accuracy_ssot.md` | Right number, right population and denominator; direction/ranking words true now; structural claims; captions; re-derived facts | No |
| R2 | `02_claims_evidence.md` | Claim strength vs evidence; causal language; over-generalising from sample; literature; attribution | Yes |
| R3 | `03_reader_experience.md` | Style-guide adherence; findings-first; residue; hedging; rendered-page artefacts | No |
| R4 | `04_statistical.md` | Analysis design **and implementation in code**: unit, normalisation scope, score comparisons, model, confounded pooling, correction variables, sensitivity scope, multiple testing | Yes |
| R5 | `05_methods_repro.md` | Methods prose matches code; parameters; versions; citations | Yes |
| R6 | `06_internal_consistency.md` | Cross-location disagreement: numbers, cross-references, conclusions, attributed quotes | No |
| R7 | `07_visual_design.md` | Figures and tables as graphics, including on-plot label placement | No |
| R8 | `08_terminology.md` | Vocabulary as used, including code keys; nomenclature; abbreviations | Yes |
| R9 | `09_taxonomy_integrity.md` | Vocabulary as defined: once, or many times | No |
| R10 | `10_constant_integrity.md` | Named values restated as literals; paths; versions | No |

### Step 4 — Collect and merge

When all 10 have reported:
- **Merge duplicates.** The same file, line and problem from more than one role becomes one
  finding, keeping the lane owner's version and noting the others.
- **Resolve handoffs.** Each `## Handoffs` line either matches a finding by the owner, or
  becomes an orchestrator-verified finding in Step 5.
- **Sanity-check reviewers' own arithmetic** against known totals. A finding whose numbers
  come from the wrong population is mis-scoped, not necessarily wrong.

### Step 5 — Verify

Before anything is shown to the user, check every **H** finding and every finding marked
`judgement`, yourself or with a verifier subagent per batch:
- open the cited file and line;
- run the cheap check, such as a grep, a small table read, or a recomputation.

Mark each finding:
- **CONFIRMED**, with the check you ran;
- **REFUTED**: drop it, and list it in an appendix with why;
- **UNVERIFIED**: keep it, with the exact check needed, and never auto-apply it.

Findings marked VERIFIED by a reviewer still get a spot-check of one in five. Record the
refutation rate per reviewer in the report. It is the calibration signal for the next run.

A finding that asks you to delete code, or to call something dead or unused, must be
confirmed by finding every caller. Deleting reachable code on a reviewer's word has
destroyed work before.

### Step 6 — Report

Write `{REVIEW_OUT_DIR}/scientific_review_report.md`:

```markdown
# Scientific Review Report
**Project / date / reviewers / targets layer / defaults assumed**

## Executive summary
3-6 sentences: the most consequential confirmed findings, and the recommended order of work.

## Statistics
| Reviewer | H | M | L | Confirmed | Refuted | Unverified |

## Fix now: verified and mechanical
[SEV][CODE] Rn — file:line — issue — exact fix

## Decide: judgement or scientific meaning
1. [SEV][CODE] Rn — file:line — issue — options A/B(/C) — recommendation

## Needs a check we could not run
[the UNVERIFIED findings, each with the exact check]

## Gate nominations
Rules that could become mechanical checks, each with its justifying incident and whether an
equivalent gate exists. Review knowledge should graduate into tooling; this list is how.

## Standards file
Entries reviewers found stale or wrong (`STD`), and new defect classes worth adding, each
with its incident.

## Refuted findings
[appendix: finding — why refuted]

## Full findings by reviewer
```

Triage rules:
- **Fix now** requires all three: CONFIRMED, `mechanical`, and no change to what a sentence
  claims.
- **Never Fix now:** anything from R2 or R4, structural changes from R3, and R9/R10
  consolidations. R9/R10 fixes touch every consumer and carry a zero-change acceptance test:
  if consolidating a definition moves a reported value, the refactor is wrong, not the value.
- Everything else goes to **Decide**.

### Step 7 — Present and stop

Show the executive summary, the statistics, and the Decide list with a one-line
recommendation per item. Give the report path. Ask for decisions (e.g. "1A, 2B"). **Stop and
wait.**

---

## MODE: REMEDIATE

1. Read the report. Proceed only with Fix-now items and the user's explicit decisions. If
   decisions are missing, present the Decide list and wait. If the report path is unknown,
   ask.
2. Assign items to remediators:

| Remediator | File | Takes |
|---|---|---|
| M1 | `remediators/01_numbers_code.md` | R1, R6, R10 fixes; values-object and code changes |
| M2 | `remediators/02_claims_methods.md` | Approved R2, R4, R5 decisions |
| M3 | `remediators/03_writing_structure.md` | R3 and R8 fixes |
| M4 | `remediators/04_figures_visuals.md` | R7 fixes, and caption fixes from R1/R3 |

3. **Parallel only over disjoint file sets.** Compute each remediator's file set first.
   Overlapping sets run one at a time, with a commit in between. Every remediator is
   forbidden `git checkout`/`restore`/`stash`/`reset`. Incident: a remediator reverted files
   it judged out of scope and destroyed another's uncommitted work. Undoing one edit with a
   checkout also reverts every other uncommitted change in that file.
4. **Verify each remediator's output yourself.** Spot-check its claims against the source.
   Rerun the project's gates. Rebuild or re-render what changed, and re-read the changed
   rendered pages. For any gate added or changed, inject the defect, confirm it fails,
   revert, and confirm it passes.
5. Write `{REVIEW_OUT_DIR}/remediation_summary.md` (applied, deferred, manual items, next
   steps) and present it.

---

## MODE: FEEDBACK

Triage collaborator comments against the codebase.

1. **Parse** into numbered items. For each, record the request, the chapters or figures
   named, and whether it is a correction ("X is wrong") or a wish ("can we show X?").
2. **Check reality.** List the chapters, figures and (targets layer) built targets, then
   classify each item:
   - already present;
   - partly present;
   - achievable from existing data;
   - needs new analysis;
   - out of scope.

   **A correction is a hypothesis to test, not a fact to apply.** Reproduce it in the code
   or data before agreeing. Collaborators are often right about the science and wrong about
   where the number came from, or the reverse.
3. **Triage** into Fix now / Decide / Deferred, using the REVIEW rules. Give effort
   estimates for achievable items. If three or more items fall in one reviewer's lane,
   offer to run that reviewer on the affected chapters.
4. **Output two drafts:**
   - the technical triage for the user;
   - a plain-language reply for the collaborator (already done, adding now, future work,
     out of scope), in the collaborator's own terms, with the numbers they will quote
     checked.

   Ask for decisions before remediating.

---

## Verifying the wiring without a full run

Run Step 1 and Step 2, then issue the 10 dispatches. Correct dispatch is enough to confirm
the wiring:
- all 10 launched;
- non-empty config;
- `TARGETS_LAYER` resolved;
- gate output attached.
