# Architecture Decision Records

This directory contains Architecture Decision Records (ADRs) for this project.

## What are ADRs?

ADRs document important architectural decisions made during development, including:
- the context and problem being solved;
- the decision made and alternatives considered;
- consequences and trade-offs.

They are the durable form a review-ratchet entry takes once a defect class or a recurring
judgement call is well-understood enough to be worth a full decision record rather than a
checklist line. See the `review-ratchet` skill's "graduate mechanical rules out" guidance
and the `adr-practice` skill for when to write one.

## ADR Index

| Number | Title | Status | Date |
|--------|-------|---------|------|
| [ADR-0001](0001-example.md) | Example decision | Accepted | YYYY-MM-DD |

Keep this table current whenever an ADR is added, or its status changes.

## Creating a new ADR

1. Copy `template.md` (from the `adr-practice` skill's `templates/adr-template.md`, or a
   local copy if one was scaffolded into this directory) to `NNNN-kebab-case-title.md`,
   using the next sequential number.
2. Fill in Context, Decision, Consequences, and Alternatives Considered in full — a
   one-paragraph ADR with no alternatives considered is usually a sign the decision wasn't
   actually contested, and a checklist entry would have been cheaper.
3. Add a row to the index table above.
4. If the ADR amends or supersedes an earlier one, say so explicitly in both documents'
   Status fields — a reader should never have to guess which of two ADRs currently applies.

## When to write an ADR, rather than a review-standards checklist entry

Write an ADR when:
- a recurring checklist entry (or several related ones) has enough shape to be a named,
  revisitable decision rather than a one-line defect — several entries collapsing into one
  ADR is a success, not bureaucracy;
- a new architectural pattern, external dependency, or major API change is being adopted;
  a performance trade-off is being made deliberately.

Keep a checklist entry (in the project's review-standards file) when the lesson is a single
concrete defect with a fix, not a decision with alternatives.

## Keeping this directory honest

A stale `Accepted` ADR describing a state the codebase moved past is worse than no ADR at
all, because it gets followed — see `adr-practice/templates/principle-claim-decay.md`. When
you find one, correct its Status field (or supersede it with a new ADR) in the same change
that revealed the drift, not as a follow-up.
