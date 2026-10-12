---
name: scientific-review
description: Run scientific review, remediation, or collaborator feedback triage on a manuscript/analysis-book project.
---

Scientific review command — see the `scientific-review` skill for full behavior.

## Usage

- `/scientific-review` — full 10-reviewer automated pass, with verification, producing a report
- `/scientific-review remediate` — apply Fix-now items; present the Decide queue for the user's decisions
- `/scientific-review feedback <text or file path>` — triage collaborator feedback against the codebase
- `/scientific-review feedback --targeted <text or file path>` — feedback triage + a focused reviewer subset on the affected chapters

## What happens

The `scientific-review` skill discovers project structure by inspection (book roots,
chapter order, R source dir, SSoT pattern, standards file, gates, and — if present — a
`{targets}` pipeline layer) rather than requiring a config file. `project_config_template.md`
is optional, for recording a repeat review's defaults only.

## Arguments

`$ARGUMENTS` is passed to the skill as the mode selector plus any additional context (e.g.
feedback text or a file path).

Mode: $ARGUMENTS
