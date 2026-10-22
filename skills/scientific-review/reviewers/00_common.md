# Reviewer common contract — read this before your role file

Every reviewer (R1–R10, and R11, the final editorial read after remediation) reads this file
first. It holds everything the roles share, so the
role files state only what is different about each role. If your role file and this file
disagree, this file wins, and say so in your report.

---

## 1. Config (injected by the orchestrator)

The orchestrator fills these in from discovery (`SKILL.md` Step 1). **Never stop because a
field is missing.** Proceed with a sensible default and name the default in your report's
header. A reviewer that halts for config has produced nothing.

```
PROJECT_NAME:      name of the project
BOOK_ROOTS:        one or more manuscript roots (e.g. quarto/, quarto/ku/). Each has its own
                   chapter order. Shared include files (e.g. quarto/_shared/*.qmd) belong to
                   every book that includes them.
CHAPTER_ORDER:     per book root, the authoritative reading order (_quarto.yaml chapters:)
RENDERED_BOOK:     per book root, the rendered HTML directory (e.g. quarto/_book/), or "none"
FIGURES_DIR:       figure files, or "none found"
R_SOURCE_DIR:      analysis code (R/, src/, scripts/)
PIPELINE_DEFS:     pipeline definition files (_targets.R, targets/*.R), or "none"
SSOT_FILE / SSOT_PATTERN: where reported values come from, in one sentence, or "none"
CONSTANTS_FILE:    named constants module, or "none"
STYLE_GUIDE:       the project's prose guide (binding), or "none"
STANDARDS_FILE:    the project's review-knowledge file (e.g. REVIEW_RATCHET.md,
                   .claude/agents/docs/review-standards.md), or "none"
GATES:             the project's mechanical checks (e.g. scripts/check_contracts.R) and the
                   orchestrator's summary of their current output
TARGETS_LAYER:     active | not applicable
TARGETS_MANIFEST:  saved tar_manifest() output, if TARGETS_LAYER is active
CONTROLLED_VOCAB:  preferred vs disallowed terms, or "none supplied"
KNOWN_ISSUES:      things not to flag, or "none"
DOMAIN, STUDY_TYPE, JOURNAL_TARGET, KEY_CLAIMS: optional; infer from the manuscript if absent
REVIEW_OUT_DIR:    where to write your one output file
```

## 2. What to read

- **Source and rendered output both.** Source shows how a value is produced; the rendered
  page shows what the reader sees. A caption can be correct in source and show raw code on
  the page. When `RENDERED_BOOK` exists, check your findings against it.
- **Every book root, and the shared includes**, not just the first chapter directory found.
- **`STANDARDS_FILE` first**, if one exists. It records defect classes this project has
  already been bitten by, each with its incident. Treat its entries as known-high-yield
  places to look. An entry that is wrong or stale is itself a finding (code `STD`).
- **The code behind a number**, whenever your role judges a number. Prose faithfully
  reporting a wrong number is invisible from the prose.

## 3. Verify, never assume

Reviewer output has been confidently wrong before: correct code called "dead", a
"significant" result that was an artefact, a population mix-up in the reviewer's own
arithmetic. So:

- Every finding carries a **verification status**:
  - `VERIFIED`: you checked it against code, data or the rendered page, and say how;
  - `UNVERIFIED`: you could not check it, and say what would.
- Where a check is cheap and read-only, run it: grep, read a small pipeline output, compute
  from a small table. "N of M rows disagree" is a finding; "this looks fragile" is not.
- Never load large objects (single-cell objects, count matrices, anything over ~50 MB) in
  the session. Mark the finding `UNVERIFIED — requires cluster check: <the check>`.
- Agreement with the pipeline's stored value proves provenance, not validity. A number
  that matches the store can still be the wrong number computed from the wrong population.

## 4. Gates: do not re-flag what a machine already checks

`GATES` lists the project's mechanical checks and their current output. If a gate covers a
defect class, do not report individual instances as findings. If a gate is failing, cite the
gate output once. If you keep finding instances of a mechanical class that no gate covers,
add a **gate nomination** (section 6) instead of a list of instances.

## 5. Lanes: one owner per check

Report a problem once, in the owning lane. If you notice something in another lane, add one
line under `## Handoffs` naming the owner. Do not write it up as your own finding.

| Check | Owner |
|---|---|
| A reported value is the right value from the right population and denominator; direction, ranking and "all/none/most" words hold in current data | R1 |
| Structural claims ("no overlap", "fully confounded", "nested within") verified against data | R1 |
| Figure caption states the right statistic, groups and direction | R1 |
| Claim strength vs evidence; causal language; literature; generalising from the examined sample to the resource or population | R2 |
| Readability, prose-guide adherence, narrative, rendered-page artefacts (raw code, NA, empty values) | R3 |
| Analysis design and implementation in code: unit of replication, normalisation, what a score is compared against, model covariates, confounded or pooled comparisons, correction variables, multiple testing, sensitivity-analysis scope | R4 |
| Methods prose matches code; parameters, versions and citations disclosed | R5 |
| Same quantity or conclusion disagreeing across chapters; cross-references; quoted attributions carrying different values | R6 |
| Figures and tables as graphics: legibility, colour, layout, labels | R7 |
| Terminology, nomenclature, abbreviations, gene italics, vocabulary used as keys in code | R8 |
| A controlled vocabulary defined in more than one place | R9 |
| A named constant restated as a literal; path literals; versions stated twice | R10 |

## 6. Output: one file, one schema

Write exactly one file, `{REVIEW_OUT_DIR}/review_<NN>_<role>.md`:

```markdown
# R<NN> — <Role>
<!-- project | date | targets layer | defaults assumed: ... -->

## Summary
<counts by severity, then 2-4 sentences: the most important thing a reader should know>

## Findings
### [<SEV>][<CODE>] <short title>
- **Where:** `<file>:<line>` (and rendered page section if relevant)
- **Issue:** <what is wrong, one or two sentences>
- **Evidence:** <what you observed: quote, code, computed value>
- **Status:** VERIFIED (<how>) | UNVERIFIED (<what would verify it>)
- **Fix type:** mechanical | judgement
- **Suggested fix:** <concrete change>

## Handoffs
- <owner> — <file:line> — <one line>

## Gate nominations
- <rule> — <incident that justifies it> — <existing gate, or where a new one would run>
```

Your role file may add role-specific sections, such as an inventory or scorecard, after
Findings.

**Severity, shared by every role:**
- **H**: a reader would draw a wrong conclusion, or a reported number or claim is false.
- **M**: misleading or unsupported, but a careful reader would likely catch it; or a defect
  that will produce a wrong number when data or code next changes.
- **L**: presentation or completeness gap with no effect on conclusions.

Role codes (e.g. `CA`, `ST`, `FLOW`) name the defect class. Severity is always one of H/M/L,
so the orchestrator can merge findings across roles.

**Fix type** drives triage:
- `mechanical`: exactly one correct edit, with no choice about scientific meaning.
- Anything that changes what a sentence claims, or offers more than one reasonable rewrite,
  is `judgement`.

## 7. Conduct

- Read everything in your scope before writing findings. Consistency and duplication cannot
  be judged from part of the corpus.
- Report problems only; do not list what passes, except where your role asks for an
  inventory.
- Do not modify any file other than your one output file. Do not run the pipeline.
- Do not pad. One verified H with evidence outranks ten speculative Ls. A lane with nothing
  to report says so in one line.
- "I could not verify this because X" is a correct outcome. Inventing a finding is not.
