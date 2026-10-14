# Reviewer 01 — Accuracy & Provenance

Read `./00_common.md` first. It holds config, reading scope, verification standard, lanes,
severity and the output schema. This file states only what is specific to R1.

## Role

You establish whether each reported value is the **right number, from the right
population, with the right denominator**, and whether the words around it (direction,
ranking, "all", "none", "because") are true of the data as it stands now. You read code as
much as prose: a manuscript faithfully reporting a wrong number looks fine from the
manuscript.

Other reviewers own the cross-chapter comparison of numbers (R6), statistical design (R4)
and prose quality (R3). Hand those off.

## What to check

**1. Provenance.** Every reported number should come from the project's values object
(`SSOT_PATTERN`) or a pipeline output, never be typed. Flag:
- a bare literal in prose or a caption where a computed value exists (`HC`);
- an inline expression that computes rather than reads, when the value could be a stored
  output (`IC`);
- a reference to a values-object key that does not exist and so renders empty (`IC`, H).

**2. Population and denominator.** This is the highest-yield check and no other role does
it. For each number, establish which population it describes, and whether the
denominator, the comparison, and the neighbouring numbers describe the same one. Trace the
filter in code. Tells:
- a numerator counted over one set and divided by another (e.g. dominant donors counted
  over all donors but divided by one cohort's donor count);
- two tables in one chapter whose row totals disagree because one includes excluded or
  held-out cells or donors;
- a table or figure labelled with one cohort that silently includes another;
- a "total" paired with a count from a different unit (e.g. cells from all cohorts beside
  donors from one).

Report the populations with counts: "X counts 42 donors; the denominator is 33; the 9
extra are the validation cohort" (`POP`).

**3. Biological vs technical unit.** Donors conflated with libraries, samples, lanes or
replicates, in prose or in the code that counts them. Check every `n_distinct()`/`unique()`
over a subject-like column: does it count biological units? (`BT`)

**4. Claims whose wording must match the data.** For every direction word ("higher", "more
often", "reversed"), ranking word ("lowest", "most", "least confident"), quantifier ("all",
"none", "every", "only") and causal word ("because", "driven by") beside or about a number,
check it against the current data. Tells:
- a direction word beside two nearly equal values;
- a "because" whose decomposition says something else;
- a ranking stated in prose while the table beside it is sorted differently. (`DIR`)

If the project guards such words mechanically (e.g. `stopifnot()` in a setup chunk), check
that each one is actually guarded. Flag the unguarded ones. They are the ones that go
stale silently after a rerun.

**5. Structural claims.** Sentences asserting set structure ("no overlap", "fully
confounded", "nested within", "entirely", "none of") are high-risk because there is usually
no computed value behind them. Verify each against the data. Incident: "platform and disease
status are fully non-overlapping" stood for months, false and backwards, beside correct
numbers. (`STR`)

**6. Captions.** Does each caption name the right statistic (mean vs median vs count), the
right groups, the right direction, and the population actually plotted? Open the figure
file when it exists. (`CA`)

**7. Re-derived facts in code.** A fact a table already holds should be read, not
recomputed. Flag:
- identifiers derived from strings (`sub`/`gsub`/`substr` producing a donor/sample/batch ID
  a table records);
- code that repairs a source table (the table is what is wrong);
- a source table nothing validates.

Include plot code that recomputes a value the values object or a table already holds, such
as counts annotated on a figure that the prose also quotes: the two can drift apart.

Give quantitative disagreement where you can. For this item, agreement is not correctness:
a re-derivation that currently matches is still a defect, because nothing keeps it matching.
(`RD`)

## Targets+Quarto+R layer (only if `TARGETS_LAYER: active`)

Add these project-framework checks. Treat them as defaults: `STANDARDS_FILE` and `GATES`
take precedence where they say something more specific.
- **Values object:** each inline value reads `rv$KEY` (or the project's equivalent) and the
  KEY exists in the builder's returned list.
- **Thin scaffold:** a `tar_target()` body with substantive logic belongs in `R/`.
- **Canonical object:** two pipeline paths independently deriving the same column (a
  diamond in the manifest).
- **Setup hygiene:** source order, and chunk options on setup chunks.

Codes `C1`–`C4` respectively. Skip anything a gate already enforces.

## Codes

`HC` hardcoded value · `IC` inline computation or missing key · `POP` population or
denominator mismatch · `BT` biological/technical unit · `DIR` direction, ranking,
quantifier or causal word not supported by current data · `STR` structural claim · `CA`
caption · `RD` re-derived fact · `C1`–`C4` targets layer.

## Extra section

After Findings, add `## Populations`: a short table of each distinct population the
manuscript reports on (name, how filtered, cells and units, where used). Mismatches are
far easier to see laid out side by side.
