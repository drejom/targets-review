# targets-review

A Claude Code plugin consolidating review philosophy, ADR practice, and R `{targets}`/Quarto
pipeline conventions for scientific (or any high-stakes) data-analysis projects — designed to
work on a brand-new project as well as an established one with its own existing conventions.

This exists because the same lessons were being independently re-learned across several
projects before being written down in any one reusable place: a review-standards "ratchet"
pattern, a library of architecture decision records, R/`{targets}`/Quarto coding
conventions, and a multi-agent review skill. This repo is the consolidation.

## What's here

| Skill | Answers |
|---|---|
| [`review-ratchet`](skills/review-ratchet/SKILL.md) | How should review knowledge accumulate in a repo instead of evaporating each session? |
| [`targets-pipeline-standards`](skills/targets-pipeline-standards/SKILL.md) | What does correct R/`{targets}`/Quarto pipeline code look like? |
| [`adr-practice`](skills/adr-practice/SKILL.md) | How do I write a durable decision record, and is there a starting library of common principles? |
| [`scientific-review`](skills/scientific-review/SKILL.md) | How do I actually run a multi-agent review pass over a manuscript/report and its code? |

Plus two commands: `/targets-review-init` (greenfield/brownfield onboarding) and
`/scientific-review` (run a review pass).

## Installation

Add this repo as a Claude Code plugin marketplace, then install the `targets-review` plugin
from it. See the Claude Code plugin documentation for the current `/plugin marketplace add`
/ `/plugin install` syntax.

## Onboarding a project

Run `/targets-review-init` in the target project. It detects whether the project is
greenfield (no pipeline code yet) or brownfield (existing code, possibly existing
conventions) and scaffolds accordingly — see the command file for the exact branching logic.
It never overwrites an existing convention; on a brownfield project, a conflict between this
plugin's recommendations and the project's own established conventions is reported, not
silently resolved in the plugin's favor.

## What's deliberately not here

Personal HPC/cluster infrastructure configuration (SSH proxy jumps, a specific lab's
Singularity container setup) is excluded — it's real operational knowledge, but it's
personal infrastructure, not portable review philosophy, and belongs in a personal
`~/.claude/CLAUDE.md` rather than a shared plugin.

Domain-specific ADRs (validating a particular statistical method, diagnosing a particular
cluster's job-hang signature) also stay in the projects that produced them — see
`adr-practice/SKILL.md` §"What doesn't belong here."

## Provenance

Consolidated from three sources: a global `~/.claude/CLAUDE.md` "Ratchet Principle" section
(the canonical statement, carried over near-verbatim into `review-ratchet`), two R
`{targets}`-pipeline projects' own `.claude/` trees (ADRs, a live review-standards document,
and project-specific style guides — genericized into the templates and reference docs here,
with project-specific incident numbers and identifiers stripped), an existing
`scientific-review` Claude Code plugin (moved in near-unchanged), and R/`{targets}`/Quarto
rigor rules supplied directly by the user and not previously written down anywhere as a
single coherent document (now `targets-pipeline-standards`' core content).
