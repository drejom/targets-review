---
name: adr-practice
description: >
  How to write an Architecture Decision Record, when a review-ratchet checklist entry
  should graduate into one, and a small starting library of genericized, pre-written
  principles (single source of truth, verify-the-premise, error-handling, claim decay,
  environment divergence, evidence quality) to copy from instead of writing from scratch.
  Triggers on "write an ADR", "should this be an ADR", "is there a template for this
  decision", "why do we have two conflicting standards", or any request to document a
  durable architectural decision.
---

# ADR practice

An ADR (Architecture Decision Record) is the durable form a review-ratchet entry takes once
it's a *decision with alternatives*, not a one-line defect with a fix. See the
`review-ratchet` skill for the accumulation philosophy this supports; this skill is the
mechanics.

## When to write an ADR vs. a checklist entry

Write an ADR when several related checklist entries share one underlying cause, or a new
architectural pattern, dependency, or trade-off is being adopted deliberately — something a
future reader needs *context and alternatives* for, not just a rule. Keep it a checklist
entry when the lesson is a single concrete defect with one fix and no real alternative was
ever on the table.

Collapsing several checklist entries into one ADR, once the cap in `review-ratchet` is
approached, is the intended outcome — not a bureaucratic afterthought. An ADR that collapses
five one-line entries into one named decision should **delete** those five lines from the
checklist and leave a single pointer in their place.

## Templates

| File | Use for |
|---|---|
| `templates/adr-template.md` | The standard ADR skeleton — copy this to start a new ADR |
| `templates/adr-readme-template.md` | A `.claude/adr/README.md` index + authoring guide, scaffolded once per project |
| `templates/principle-ssot.md` | Every fact has exactly one home — the single-source-of-truth test and its four defeating properties |
| `templates/principle-verify-premise.md` | Verify a premise before implementing against it — four recurring sources of false premises |
| `templates/principle-trycatch.md` | Error handling must not collapse distinguishable failures into one silent fallback |
| `templates/principle-claim-decay.md` | A written claim decays until re-verified — derive it, date it, or re-check it before acting |
| `templates/principle-environment.md` | The environment that tests is not the environment that runs — name the environment when reporting a pass |
| `templates/principle-evidence-quality.md` | Count sources, not checks; a failing control invalidates the method |

Each principle template is genericized from real, independently-converged-on incidents
across more than one project, with the project-specific details (issue numbers, internal
identifiers) stripped and replaced with one tight anonymized example. Copy one into a
project's `.claude/adr/` directory and adapt the worked example to the project's own
incident when one arises — don't just cite the template verbatim forever; a project-specific
incident is what makes the rule land.

These six cover the principles that recurred across more than one source project. They are
a starting library, not an exhaustive one — a project will accumulate its own domain-specific
ADRs over time (see "What doesn't belong here" below).

## Graduating a rule into a mechanical gate

The review-ratchet's "graduate mechanical rules out" duty means: once several checklist
entries describe something a script could check, write the script and replace the prose with
a one-line pointer to it. A minimal illustration of the shape (adapted from a project that
enforces several such gates in one script):

```bash
# Mechanically enforce review-standards entries that CAN be mechanised.
# Every check below started life as a checklist entry with a real incident behind it.

fail=0

report() {
    fail=1
    printf '\n\033[31mFAIL\033[0m %s\n' "$1"
    shift
    printf '  %s\n' "$@"
}

# Example: flag a banned pattern with the incident that justified banning it
hits=$(grep -rn "some_banned_pattern" src/*.* 2>/dev/null || true)
if [ -n "$hits" ]; then
    report "banned pattern found (see ADR-00NN for why)" "$hits"
fi

exit $fail
```

The pattern that matters: each check's own header names the incident that justified it, so
the *gate* carries the evidence, not just the checklist entry it replaced. A reader asking
"why does this check exist" finds the answer at the check, not in a separate document that
can drift out of sync with it.

## What doesn't belong here

Domain-specific ADRs — validating a particular statistical method for a particular dataset,
diagnosing a specific class of job hang on a specific cluster, a naming convention tied to
one project's metadata schema — stay in the project that produced them. They're often
excellent ADRs; they just don't generalize, and vendoring them into a shared plugin would
make this skill a second dumping ground for project-specific knowledge, which is exactly
what `review-ratchet`'s scoping rule warns against.

## Related skills in this plugin

- **`review-ratchet`** — the philosophy this skill operationalizes.
- **`targets-pipeline-standards`** — R/`{targets}`/Quarto-specific conventions; several of
  its rules (e.g. "never infer structural facts from naming conventions when an authoritative
  source exists") are specializations of `principle-ssot.md` and `principle-verify-premise.md`
  for that stack specifically.
