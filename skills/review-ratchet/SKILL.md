---
name: review-ratchet
description: >
  How review knowledge should accumulate in a repository instead of evaporating at the end
  of each session. Read this before setting up a review-standards file for a project, before
  judging whether an existing one is stale, or before deciding where a newly-learned lesson
  should live. Triggers on "set up a review standards file", "where should this lesson go",
  "is this project's review-ratchet file stale", "add this to the project's standards", and
  is read automatically by `/targets-review-init`. Applies to any project in any language —
  the R/{targets}-specific content lives in the sibling `targets-pipeline-standards` skill.
---

# The Ratchet Principle — review knowledge accumulates in the repo, not the session

**Applies to every project, in any language.**

Each repo keeps ONE living document of its review knowledge — conventionally
`.claude/agents/docs/review-standards.md` (or `REVIEW_RATCHET.md` at the repo root) —
referenced as a **standing order** from whatever file agents always read (`CLAUDE.md`,
`AGENTS.md`, etc). Any agent that implements, reviews, or lands a change reads it first.

If the repo has no such file yet and you hit a defect class worth remembering, create one.
Starting it is cheap; rediscovering the same trap is not.

## Why

- **Automated gates pass while the change is wrong.** Green tests answer "did I break what
  I already understood?", never "is this the right change?". A repo with thousands of
  passing tests has shipped a quoting bug that made a provenance field silently record
  "unavailable".
- **Reviewers get bypassed by timing.** A bot that comments *after* a PR opens is invisible
  to a workflow that merges on open. Know when your reviewers post.
- **Stale guidance actively misleads.** A doc marked `Accepted` described a state reverted
  months earlier and derailed a whole design session. **A wrong standard is worse than
  none, because it gets followed.**

## The three duties — discharge them in the same PR as the triggering work

| Duty | Rule |
|---|---|
| **Add** | New defect class found → one-line entry **plus the concrete incident**. No incident = speculation = delete on sight. |
| **Correct** | Entry wrong or outdated → fix or remove it, and say so in the commit. |
| **Deduplicate** | Check before adding. Two phrasings of one rule is how contradictions begin. |

Plus a **two-minute hygiene pass before you start**: scan for entries that contradict each
other or the repo's other docs; spot-check two or three against current code; consolidate
if it has sprawled. Report what you changed, so the ratchet is visible rather than silent.

## Rules that keep it from rotting

- **Cite the incident, always.** It lets a future reader judge whether an entry still
  applies instead of obeying it blindly, and it filters out speculation.
- **The correction duty is the load-bearing half.** Append-only lists rot into the exact
  failure they were meant to prevent. This is where such schemes fail.
- **Cap it at roughly 150 lines of checklist.** Past that nobody reads it, and an unread
  standard is a stale one. Consolidate, or promote a recurring theme into a proper decision
  record (ADR — see the sibling `adr-practice` skill) and link it.
- **Graduate mechanical rules out.** If a rule can become a lint check or a test, make it
  one and have the doc point at the command. The document is for judgement that cannot be
  automated *yet*; it should shrink as tooling catches up.
- **Scope it to review knowledge only.** Implementation conventions, domain vocabulary, and
  one-off decisions belong in their own files, or this becomes a second dumping ground.

## Universal section headings (entries differ per stack; headings do not)

1. **The gate sequence** — what must happen before merge, in order, including who reviews
   and *when they actually post*
2. **Never-touch list** — files or actions unsafe in specific contexts, with the reason
3. **Defect checklist** — grouped by category, every entry earned
4. **Verify, never assume** — the standing epistemic rule

## The framing that makes it stick

> Reporting "I could not satisfy this, because X" is a correct outcome.
> Inventing work to fill a ticket is not.

And: **a reviewer is not an oracle.** If a finding is wrong, say so with evidence rather
than complying — a wrong "fix" is worse than a declined comment. Audits and review bots
produce confidently-wrong findings; one called correct code "dead", and deleting it would
have destroyed real work. Verify a finding against the code before acting on it —
including findings in the standards doc itself.

## Rule 0 — a project skill can be self-improving

If a project has its own `CLAUDE.md`-adjacent skill (project architecture, domain
vocabulary, local conventions — distinct from this plugin's portable content), apply the
same ratchet logic to it directly, in the same turn the drift is noticed:

- **Error discovered** → fix the relevant file immediately, before moving on. Do not defer.
- **New fact learned** (an architecture decision, a resolved bug, a new helper, a changed
  path) → record it right away.
- **Something found to be outdated** → update or remove it on the spot.
- **Drift noticed in passing** (a function renamed, a constant moved) → correct it without
  waiting to be asked.
- **End of a session or a piece of work** → a brief retrospective: scan what changed and
  update anything that no longer reflects reality.

The goal is that any future session reading the project's own skill gets an accurate
picture without needing the full conversation history. When in doubt, write it down.

## Reference implementations

Two projects independently converged on this pattern before it was written down here —
worth reading as worked examples, each with a structural variant worth borrowing:

| Project | File(s) | Notable variant |
|---|---|---|
| haemProcessR | `.claude/agents/docs/review-standards.md`, first formalized in its own ADR-0016 | A **mechanical gate** (`check-repo-hygiene.sh`) enforces the 150-line cap itself, rather than leaving the count to memory — see `adr-practice/SKILL.md` for the excerpt and why. |
| A second R/{targets} project | `REVIEW_RATCHET.md` | States the duties as **four** (Add/Correct/Deduplicate/**Graduate**) rather than three, with Graduate broken out as its own numbered duty rather than folded into "rules that keep it from rotting." Worth adopting if your project graduates rules often enough that it deserves top billing. |

Both arrived at nearly identical wording for "a wrong standard is worse than none" and "a
reviewer is not an oracle" independently — treat convergent, independently-derived wording
as a signal the rule is real, not an artifact of one project's house style.

## Related skills in this plugin

- **`targets-pipeline-standards`** — what correct R/`{targets}`/Quarto code looks like. The
  ratchet tells you *how* review knowledge should accumulate; this skill is the starting
  content for an R/{targets} project's own defect checklist.
- **`adr-practice`** — how to capture a single durable decision the ratchet's "promote to an
  ADR" step produces, with a template and a small library of genericized, pre-written
  principles (single source of truth, verify-the-premise, etc.) to start from instead of
  from a blank page.
- **`scientific-review`** — how to actually run a review pass over a manuscript and its
  code. The ratchet is about accumulating knowledge between reviews; this skill is the
  review itself.
