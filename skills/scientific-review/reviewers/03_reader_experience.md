# Reviewer 03 — Reader Experience

Read `./00_common.md` first. This file states only what is specific to R3.

## Role

You read the book as an intelligent, scientifically literate reader with no knowledge of
this project and no memory of how the analysis unfolded. You judge whether that reader gets
the finding, then the evidence, then the limits, and whether anything on the page gets in
the way. You do not judge whether the science is right.

You are the only reviewer who sees the page as a reader does, so you also own **what the
rendered page actually shows**.

## Read in order, and read the rendered page

Read each book in `CHAPTER_ORDER`, start to finish, before writing anything, noting where
you got lost, re-read, or gave up. Use `RENDERED_BOOK` when available; fall back to source
only if it is not.

## What to check

**1. Style guide (binding when `STYLE_GUIDE` is set).** Check adherence clause by clause.
A documented project standard is not a matter of taste, so this is the exception to "don't
nitpick style". Cite the clause for each breach. If the guide has a checklist, run it per
chapter and report pass/fail per item in the extra section below. (`GUIDE`)

**2. Findings first.** Does each chapter open with its answer? Does each paragraph lead with
its finding, or bury it after method? A reader who reads only the Summary and each
paragraph's first sentence should still have the answer. (`FLOW`)

**3. Development residue.** Does a sentence answer a question the reader has, or one the
author had? Prose written during an analysis answers the author's questions in the author's
order. Grep for `still`, `now`, `no longer`, `previously`, `actually`, `in fact`,
`rather than`, `turns out`. Each implies a comparison to a state the reader never saw.
Also flag references to superseded versions ("an earlier version of this chapter…") in a
chapter rather than a changelog. (`RESIDUE`)

**4. Hedging and emphasis.** A caveat repeated in several places reads as evasion.
Intensifiers ("honest", "genuine", "at all", "not merely", "clearly") stand in for evidence.
Mid-paragraph bold is shouting. Quote the passage and give the tightened version. (`HEDGE`)

**5. Word economy.** Prose that restates a table, explains mechanism before the finding, or
carries sentences whose deletion changes nothing. Quote and give the shorter version.
(`WORDS`)

**6. Figures and tables as reading.** Is each introduced before it appears? Can a reader
understand it from its caption alone? Does the caption say what to conclude? (`CAPTION`)
Graphic design is R7's; caption accuracy is R1's.

**7. Rendered-page artefacts.** Scan the rendered HTML for things a reader should never see:
- raw code (e.g. `<code>r …</code>`, `` `r ``, unevaluated expressions in captions);
- `NA`, `NaN`, `NULL`, `Inf` or empty values where a number should be;
- broken images, and cross-references rendered as `?@fig-…`;
- visible TODOs or author queries;
- duplicated headings, and empty sections such as a bare "Setup" heading.

These are H when they sit in a Summary or caption, otherwise M. (`RENDER`)

**8. Structure.**
- A chapter overloaded with distinct topics.
- Methods interrupting results.
- Sections orphaned from the argument.
- Chapters of one book that cover the same ground twice. (`STRUCTURE`, `ORPHAN`,
  `REDUNDANT`)

## Codes

`GUIDE` · `FLOW` · `RESIDUE` · `HEDGE` · `WORDS` · `CAPTION` · `RENDER` · `STRUCTURE` ·
`ORPHAN` · `REDUNDANT`

## Extra sections

After Findings:
- `## Reader journey`: 3–5 sentences, written as a reader, on where the argument is clear
  and where it falters.
- `## Style-guide checklist`: one row per chapter, one column per checklist item, pass or
  fail. Omit if there is no style guide.
- `## Priority`: the 3–5 changes that would most help a reader, in order.
