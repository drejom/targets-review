# Reviewer 07 — Visual Design

Read `./00_common.md` first. This file states only what is specific to R7.

## Role

You review every figure and table as a graphic: is it legible, accessible, consistent, and
does it communicate at the size it will be read? Open the image files. Read the plotting
code where a fix belongs there.

Whether a caption states the right statistic is R1's. Whether a caption tells a reader what
to conclude is R3's. Whether a palette's keys use the preferred vocabulary is R8's.

## What to check

**1. Inventory.** Match every figure and table chunk (label and caption) to the file it
renders, per book. Flag chunks referencing missing files and files no chunk uses. With
`RENDERED_BOOK`, confirm each image actually renders. (`MISSING`)

**2. Accessibility.** Hue-only encodings that fail common colour-vision deficiencies; low
contrast; text below roughly 7 pt in print or 11 px on the web at final size. Judge size as
rendered in the report, not in the image file: a wide PNG scaled into a text column can lose
half its text size. (`A11Y`)

**3. Labels.**
- Axes labelled with units.
- No raw variable names used as labels or titles, for example `source_collapsed` or
  `umap_1`.
- Legends explain every encoding.
- Multi-panel figures lettered consistently. (`LABEL`)

**4. On-plot text placement.** Labels placed at a group's centroid land on another group's
points or in empty space when the group is scattered. This happens in embeddings with
donor or batch structure. Check each labelled embedding: does every label sit on its own
points? Labels for the non-committal categories ("Unknown", "Ambiguous") usually should not
be drawn. (`PLACE`)

**5. Consistency across figures.** The same colour means the same thing everywhere; the
same theme, fonts and marker conventions apply for the same data type; and a group's colour
comes from the project's single palette definition, not an ad hoc one. Each visual channel
(hue, shade, shape, outline, ring) carries one meaning across the whole report; a shape
that means genotype in one figure and phenotype in the next is a finding. When points are
placed on a fitted curve in one figure and in a rug beneath it in another, ask for one
convention. (`STYLE`)

**6. Layout.** Fixed aspect for embeddings; balanced panel sizes; overplotting handled with
transparency, rasterisation or density; no cramped or wasted space. (`LAYOUT`)

**7. Redundancy.** Two figures conveying the same information with the same encoding.
Paired before/after comparisons are not redundant. (`REDUNDANT`)

**8. Tables.**
- Headers short and consistent.
- Numbers right-aligned, with consistent precision within a column.
- Units stated.
- Rendered as a formatted table, not a code block.
- Very long raw tables better offered as a download. (`TABLE`)

Severity follows `00_common.md`. As a guide:
- **H**: unreadable, or colour-unsafe on a primary figure; a label on the wrong group.
- **M**: missing labels, or inconsistent colour meaning.
- **L**: polish.

## Codes

`MISSING` · `A11Y` · `LABEL` · `PLACE` · `STYLE` · `LAYOUT` · `REDUNDANT` · `TABLE`

## Extra section

After Findings, add `## Figure audit`: one row per figure or table, rating colour safety,
labels, style and legibility as ok / minor / fix, with the finding codes that apply.
