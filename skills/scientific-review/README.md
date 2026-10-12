# scientific-review

A multi-agent scientific review skill for manuscripts/analysis books (Quarto, R Markdown,
LaTeX, or plain markdown) and the analysis code behind them. Ported into the
`targets-review` plugin unchanged in behavior; this file keeps the design history that
explains why the skill is shaped the way it is.

## Design history

**Auto-discovery over config.** The skill infers project structure (`BOOK_ROOTS`,
`CHAPTER_ORDER`, `R_SOURCE_DIR`, `SSOT_FILE`, `STANDARDS_FILE`, `GATES`, etc.) by
inspection rather than requiring a filled-in config file first. `project_config_template.md`
exists only to let a project *record* a repeat review's defaults — it is optional, never a
prerequisite.

**Targets+Quarto+R is a detected layer, not a requirement.** Step 1b of `SKILL.md` checks
for `_targets.R`/`targets/*.R` and, if found, adds `tar_manifest()`-based checks
automatically. On a project without `{targets}`, those sections are skipped outright, not
attempted — the skill works on any manuscript/code pairing.

**Ten reviewer roles, added incrementally as gaps were found in practice:**
- The original eight (accuracy/SSoT, claims/evidence, reader experience, statistical,
  methods reproducibility, internal consistency, visual design, terminology) covered
  whether a reported *value* traces to its source, whether prose *states* the parameters
  code uses, whether chapters *disagree*, and whether vocabulary is *used* consistently.
- **R9 (taxonomy integrity) and R10 (constant integrity) were added** because none of the
  above asked whether a definition exists in exactly one place. A controlled vocabulary
  (enumerable membership — R9) and a named constant (semantic identity, not just textual
  match — R10) are different judgement calls, so they're split rather than merged; R10
  cross-references R9 rather than restating it. Neither may auto-apply a fix: consolidating
  a definition touches every consumer, so both land in the Decide queue with a zero-change
  acceptance test — if consolidating where something is defined changes a reported value,
  the refactor is wrong, not the value.
- A later restructuring moved shared material (config, reading scope, the verification
  standard, the gate rule, the lane table, the finding schema) into `reviewers/00_common.md`
  so each role file states only what's specific to it — per-role config blocks had drifted
  out of sync with each other and with the "never demand config" rule.

**Reviewers are evidence, not oracles.** Every H-severity or judgement-type finding is
verified by the orchestrator (`SKILL.md` Step 5) before it reaches the user — confirmed,
refuted, or marked unverified with the exact check needed. This exists because unverified
reviewer output has been confidently wrong in practice: calling correct code "dead", calling
an artifact "significant", or mis-scoping a population in the reviewer's own arithmetic.

**Triage: "Fix now" requires CONFIRMED and mechanical.** Everything else — anything
touching what a sentence claims, or offering more than one reasonable rewrite — goes to
Decide. An earlier version auto-applied whole reviewer categories, which would have
auto-applied population/denominator errors alongside genuine typos.
