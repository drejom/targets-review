# Changelog

## 0.1.0 — initial consolidation

Initial release, consolidating:
- the "Ratchet Principle" from a personal `~/.claude/CLAUDE.md`, into `review-ratchet`;
- ADR practice and six genericized principle templates (single source of truth,
  verify-the-premise, error-handling/tryCatch, claim decay, environment divergence,
  evidence quality), derived from two R `{targets}`-pipeline projects' own ADR directories,
  into `adr-practice`;
- R/`{targets}`/Quarto coding and report-authoring conventions, derived from a project style
  guide and primary-source `{targets}` package research, plus user-authored pipeline-rigor
  rules not previously written down elsewhere, into `targets-pipeline-standards`;
- an existing `scientific-review` local Claude Code plugin, moved in unchanged.

Two commands: `/targets-review-init` (new) for greenfield/brownfield onboarding, and
`/scientific-review` (moved, lightly corrected to match its skill's actual "discover, don't
configure" design).
