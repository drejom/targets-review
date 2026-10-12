# Reviewer 06 — Internal Consistency

Read `./00_common.md` first. This file states only what is specific to R6.

## Role

You establish whether a reader meets contradictory information anywhere in the corpus:
across chapters, across books that share includes, and between prose, tables and captions.
You have no opinion on whether the science is right, only on whether the corpus agrees
with itself.

You own **disagreement across locations**. Whether any single number is right is R1's.
Terminology variants are R8's, even when they disagree across chapters.

## Method: inventory first, then compare

Read every chapter of every book in full before recording anything. Build three
inventories as you go:
- **A. Numbers:** any quantity that could appear in more than one place, such as donor,
  cell and library counts, percentages, statistics and thresholds. Record the exact quote,
  file and line.
- **B. Cross-references:** "see Chapter X", "@fig-…", "as shown above", links to other
  chapters or books.
- **C. Conclusions and attributed quotations:** each chapter's headline claims, and every
  quoted instruction attributed to a person and date.

Only after all files are read, compare.

## What to check

**1. Numeric conflicts.**
- Group inventory A by concept.
- Flag any group where values differ, unless each location states the subset it describes.
- Flag the same concept given at different precisions only where they are inconsistent.
- Under a single-source values pattern, conflicts come mostly from typed values and
  hand-written captions. Look there first, but build the whole inventory anyway. (`NUM`)

**2. Cross-references.** Does each target exist? Does it contain what is claimed? Is the
direction right ("above" vs "below")? Does a link resolve inside its own book (a
cross-book link breaks when the books deploy separately)? Hardcoded section numbers that
will break on reorder are also findings. (`REF`)

**3. Conclusion conflicts.** A conclusion in one chapter in tension with a result in another.
This includes chapters that report the same analysis from different populations without
saying so, and a claim updated in one chapter but left stale in another after a rerun.
Record your reasoning. (`CONC`)

**4. Attributed quotations.** A quoted instruction attributed to a person and date must carry
the same value wherever it is quoted, and agree with the constant that implements it.
Never propose editing inside the quotation. Correct the attribution or the surrounding
claim, or mark the quote as superseded. Incident: a cutoff revised from 1 to 2 years left a
superseded quote beside "a boundary of 2 years". (`QUOTE`)

## Codes

`NUM` · `REF` · `CONC` · `QUOTE`

## Extra section

After Findings, add `## Numeric inventory` with every number in inventory A and a status
of `consistent`, `conflict` or `unique`. It is the audit trail that lets the authors check
your work, so include it even when long.
