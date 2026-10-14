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
contrast; text below roughly 7 pt in print or 11 px on the web at final size. (`A11Y`)

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
comes from the project's single palette definition, not an ad hoc one. (`STYLE`)

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

**9. One meaning per mark.** Every visual channel (hue, shade, shape, outline,
ring, size) carries exactly one meaning across the whole report, and an
annotation mark (a ring, an outline, a star) marks one condition only. A plot
of relabelled records should draw the label the analysis uses and mark only
where it differs from the source record; mixing the two makes the reader guess
which label a point shows. Incident: a ring meant "SNP family differs from the
recorded clutch" in one figure and "label inferred" in another until it was
pinned to one rule. (`ENCODE`)

**10. Positions that are not data.** Vertical jitter on a one-dimensional
strip, or points stacked under a density curve, invent a coordinate. When
points sit on a curve (a density or potential landscape), their height is the
curve, not a measurement: they must actually touch the curve they claim to sit
on (check slopes and rescaled panels, where a vertical offset floats or sinks),
and the caption must say what height means. Incident: embryos drawn in a row
beneath each day's landscape read as a second variable; on steep walls,
vertically offset points floated off the curve. (`POSITION`)

**11. Rendered size.** Judge legibility in the rendered book at its column
width, not in the source image. A figure placed in a narrow body column
shrinks every label; widen the column (`column: body-outset` or `page`) or
redraw. (`SIZE`)

Severity follows `00_common.md`. As a guide:
- **H**: unreadable, or colour-unsafe on a primary figure; a label on the wrong group.
- **M**: missing labels, or inconsistent colour meaning.
- **L**: polish.

## Codes

`MISSING` · `A11Y` · `LABEL` · `PLACE` · `STYLE` · `LAYOUT` · `REDUNDANT` · `TABLE` ·
`ENCODE` · `POSITION` · `SIZE`

## Extra section

After Findings, add `## Figure audit`: one row per figure or table, rating colour safety,
labels, style and legibility as ok / minor / fix, with the finding codes that apply.
