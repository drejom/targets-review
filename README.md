# targets-review

A Claude Code plugin: review philosophy, ADR practice, and R `{targets}`/Quarto pipeline
conventions for scientific (or other high-stakes) data-analysis projects. Works on a new
project or an established one with its own existing conventions.

## Skills

| Skill | Use it to answer |
|---|---|
| [`review-ratchet`](skills/review-ratchet/SKILL.md) | How should review knowledge accumulate in this repo instead of evaporating each session? |
| [`targets-pipeline-standards`](skills/targets-pipeline-standards/SKILL.md) | What does correct R/`{targets}`/Quarto pipeline code look like? |
| [`adr-practice`](skills/adr-practice/SKILL.md) | How do I write a durable decision record, and what's a reasonable starting template? |
| [`scientific-review`](skills/scientific-review/SKILL.md) | How do I run a multi-agent review pass over a manuscript/report and its code? |

Read `review-ratchet` first in any project — the other three skills assume it. Each skill's
own frontmatter lists its trigger phrases; load the skill when a task matches rather than
waiting to be told.

## Commands

- `/targets-review-init` — run once per project. Detects greenfield vs. brownfield and
  scaffolds a review-knowledge file, an ADR directory, and the recommended `R/`/`targets/`
  layout accordingly. Never overwrites an existing convention; on a brownfield project, a
  conflict between this plugin's recommendations and the project's own conventions is
  reported for the user to decide, not resolved in the plugin's favor.
- `/scientific-review` — run a review pass (also `remediate` and `feedback` modes).

## How the skills relate

`review-ratchet` is the accumulation mechanism — it doesn't tell you what good code looks
like, only how review knowledge should be captured once you find a defect.
`targets-pipeline-standards` is the domain content that mechanism accumulates around for an
R `{targets}` project. `adr-practice` is how a recurring checklist entry graduates into a
durable, revisitable decision. `scientific-review` is the one skill that actually executes a
review, using the project's own accumulated review-knowledge file (found by discovery, not
configured) alongside these standards. Don't restate one skill's content inside another —
cross-reference it.
