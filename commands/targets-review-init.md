---
name: targets-review-init
description: Onboard the current project onto the targets-review system — detects greenfield vs. brownfield and scaffolds accordingly.
---

Onboard the current project directory onto `targets-review`. Read the `review-ratchet`,
`targets-pipeline-standards`, and `adr-practice` skills before proceeding if you haven't
already in this session.

## Step 1 — Discover

Check, in order:
1. Does `_targets.R` or `targets/*.R` exist? Does `R/` exist with files in it?
2. Does a review-knowledge file already exist — `REVIEW_RATCHET.md`,
   `.claude/agents/docs/review-standards.md`, or anything a project `CLAUDE.md`/`AGENTS.md`
   names as its standing order?
3. Does `.claude/adr/` (or an equivalent ADR directory) already exist?
4. Does the project have its own style guide or coding-convention doc already (a project
   `CLAUDE.md`, a `references/*.md` under an existing project skill, a `CONTRIBUTING.md`)?

Report what you found before doing anything else.

## Step 2 — Branch on what you found

### Greenfield (no `_targets.R`, no populated `R/`)

Offer to scaffold:
- The recommended layering skeleton from `targets-pipeline-standards/references/r-coding-conventions.md`
  §4: `R/constants.R`, `R/utils.R`, `R/plots_*.R`, `R/report.R`, `targets/`, `quarto/`.
- A review-knowledge file (`.claude/agents/docs/review-standards.md` is a reasonable
  default path) seeded with the four universal section headings from `review-ratchet`
  (gate sequence / never-touch list / defect checklist / verify-never-assume) — empty
  sections are fine; they fill in as real incidents happen.
- `.claude/adr/` containing `adr-practice/templates/adr-template.md` and
  `adr-readme-template.md`, copied in (not symlinked — a project should be able to edit its
  own copy freely).
- One standing-order line in the project's `CLAUDE.md`/`AGENTS.md` (create the file if
  absent) pointing at the review-knowledge file, in the spirit of `review-ratchet`'s own
  "Applies to every project" framing — don't copy the whole Ratchet Principle text into the
  project's own `CLAUDE.md`; a one-line pointer plus "this plugin is installed" is enough,
  since the full content already lives in this plugin's skills.

Confirm each scaffolding action with the user before writing — this command creates files
in a project that didn't ask for them yet, which is different from the advisory-only
behavior of the other skills in this plugin.

### Brownfield (existing `R/`, `_targets.R`, possibly its own conventions)

**Never overwrite anything found in Step 1.**

- If an existing review-knowledge file was found: do nothing to it except note it, and treat
  it as `STANDARDS_FILE` the way `scientific-review`'s own discovery step already does.
- If the project's own conventions (namespace style, layering, test patterns) conflict with
  `targets-pipeline-standards`' recommendations — e.g., the project doesn't use `pkg::fn()`
  namespacing, or organizes `R/` differently — **report the conflict explicitly and let the
  user decide** whether to adopt, partially adopt, or keep the project's own rule. Do not
  silently impose this plugin's conventions on an existing codebase; "a wrong standard is
  worse than none" applies to this plugin's own advice too, if it doesn't fit the project.
- Offer ADR scaffolding (`.claude/adr/` + templates) only if no ADR directory already exists.
- If the project has no review-knowledge file at all, offer to create one — this is additive
  (nothing to conflict with) and is the single highest-value thing this command can do for
  an established project with no accumulated review knowledge yet.

## Step 3 — Surface what `scientific-review` would find

Either path, finish by running (or describing, if the user prefers not to actually invoke
it yet) `scientific-review`'s own Step 1 discovery — book roots, chapter order, R source
dir, SSoT pattern, gates — so the user sees what a review pass would see, without launching
a full review. This also validates that the onboarding didn't miss anything `scientific-review`
would otherwise need.
