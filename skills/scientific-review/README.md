# scientific-review

A multi-agent scientific review skill for manuscripts/analysis books (Quarto, R Markdown,
LaTeX, or plain markdown) and the analysis code behind them.

## Why it's shaped this way

**Auto-discovery over config.** The skill infers project structure (`BOOK_ROOTS`,
`CHAPTER_ORDER`, `R_SOURCE_DIR`, `SSOT_FILE`, `STANDARDS_FILE`, `GATES`, etc.) by inspection.
`project_config_template.md` only lets a project *record* a repeat review's defaults — never
a prerequisite. Never stop a review to ask for a filled-in config.

**`{targets}`+Quarto+R is a detected layer, not a requirement.** `SKILL.md` Step 1b checks
for `_targets.R`/`targets/*.R` and, if found, adds `tar_manifest()`-based checks
automatically. On a project without `{targets}`, skip those sections outright rather than
attempting them — the skill works on any manuscript/code pairing.

**Ten reviewer roles, each with a single owning lane** (`reviewers/00_common.md` §5), so a
problem is reported once:
- R9 (taxonomy integrity) and R10 (constant integrity) are split because they're different
  judgement calls: a controlled vocabulary has enumerable membership (R9); a named constant's
  duplication is a semantic-identity question, not just a textual match (R10). R10
  cross-references R9 rather than restating it.
- Neither may auto-apply a fix: consolidating a definition touches every consumer, so both
  land in the Decide queue with a zero-change acceptance test — if consolidating where
  something is defined changes a reported value, the refactor is wrong, not the value.

**Reviewers are evidence, not oracles.** Every H-severity or judgement-type finding is
verified by the orchestrator (`SKILL.md` Step 5) before it reaches the user — confirmed,
refuted, or marked unverified with the exact check needed. Unverified reviewer output can be
confidently wrong: calling correct code "dead", calling an artifact "significant", or
mis-scoping a population in the reviewer's own arithmetic. Verify before surfacing.

**Triage: "Fix now" requires CONFIRMED and mechanical.** Everything else — anything touching
what a sentence claims, or offering more than one reasonable rewrite — goes to Decide. Never
auto-apply a whole reviewer category; that would auto-apply population/denominator errors
alongside genuine typos.
