# `targets` package semantics — primary-source reference

Grounded in the `targets` R package's own documentation (the manual at
https://books.ropensci.org/targets/ and `?tar_target`/`?tar_read`/`?tar_deps`/
`?tar_option_set`/`?tar_outdated`/`?tar_group` reference pages), not blog posts or inference.
Where a claim below has no clear primary-source statement, it's flagged as such rather than
asserted — do the same when extending this document.

## 1. Static dependency detection

`targets` detects a command's dependencies via **static code analysis**
(`codetools::findGlobals()` under the hood), not by running the code. It "searches commands
and functions for dependencies, noting global symbols... and ignoring local symbols."
`tar_deps(expr)` runs this analysis directly so you can inspect what a command's dependencies
resolve to.

**What defeats static analysis:**
- **Namespaced calls (`pkg::fun()`) are not registered as dependencies on `fun()`.** This is
  an explicit, named limitation: `::` is parsed as a function call with arguments `package`
  and `fun`, not as a reference to `fun` itself. `tar_target(out, fun())` registers a
  dependency on `fun`; `tar_target(out, pkg::fun())` does not.
- **Functions from attached-but-not-`imports`-registered packages are not tracked at all by
  default** (see §6).
- Character-string target names, `get()`, or `eval()` inside a command are **not explicitly
  addressed** by the primary sources — reasoning by extension from "static code analysis"
  makes it plausible such patterns aren't recognized as symbolic references, but treat this
  as inference, not a documented guarantee, until checked against the installed version.

## 2. Never call store-reading functions from inside a running target

`tar_make()`, `tar_read()`, `tar_load()`, `tar_meta()`, `tar_progress()`, and `tar_outdated()`
read or modify the pipeline's local data store. The docs state explicitly: **"the local data
store is in flux while a pipeline is running... so please do not call these functions from
inside a target as part of a running pipeline."** The only documented exception is literate-
programming target factories in `tarchetypes` (`tar_render()`, `tar_quarto()`).

This matters most for the common anti-pattern of reading another store's target
(`tar_read_raw(name, store = OTHER_STORE)`) from inside a target's own command body — it
both violates this rule and forgoes the dependency-graph visibility described in §3.

## 3. The sanctioned cross-project bridge is a `format = "file"` target

For reading one project's pipeline output from another project, the manual (ch. 8
"Projects," §8.4.2 "Sharing targets") gives two approaches:

1. **Global-object approach** (simple, can be inefficient) — directly read/load the upstream
   project's target value.
2. **File-based bridge** (recommended for large data) — declare a target in project B that
   points at a stable path inside project A's `_targets/objects/` store, with
   `format = "file"`:
   ```r
   tar_target(file_from_project_a, "store_a/objects/target_name", format = "file")
   ```

This is the documented, sanctioned pattern. An ad hoc `tar_read_raw(name, store = ...)` call
inside a target body gets neither the "don't call store-reading functions from inside a
running target" safety (§2) nor the dependency-graph auto-invalidation this pattern gives —
because an ad hoc call like that is invisible to `tar_outdated()`/`tar_visnetwork()`, not
being a symbol reference caught by static analysis (§1).

## 4. `format = "file"` semantics

- The target's command must return a character vector of **paths that exist on disk** when
  the target finishes, or `character(0)` if nothing was created. This is not optional:
  `targets` throws an error immediately if the returned path doesn't exist.
- If `repository` is not `"local"` (cloud storage), the returned vector must have length 1.
- `tar_option_set(trust_timestamps = TRUE)` (the default) means `targets` checks the file's
  timestamp first; if it agrees with recorded metadata, the file is considered unchanged,
  and the content hash is only recomputed when timestamps disagree. So the format is
  fundamentally content-hash-based, with an mtime fast-path layered on top.
- Behavior when a previously-tracked file is moved or deleted between runs is not explicitly
  documented beyond "the path must exist when the target finishes" — don't assume a specific
  recovery behavior without testing it against the installed version.

## 5. Reading a dynamically-branched (`pattern = map(...)`) target

`tar_read()` on a pattern, without specifying `branches`, **automatically aggregates every
branch's output** according to the target's `iteration` mode — it is not returned as a list
keyed by branch name by default. Use the `branches` argument (`tar_read(x, branches = 1)`) to
read one branch specifically.

## 6. Package-function invalidation requires explicit opt-in

**By default, `targets` does not track changes to functions defined inside an attached or
installed package** — only sourced/global-environment functions are tracked automatically.
The documented quote: *"targets does not track package functions by default because this is
not a common need."*

The opt-in is `tar_option_set(imports = "yourpackage")`, which tells `targets` to dive into
that package's namespace and track its objects as if they were in the global environment,
with four documented caveats:
1. Namespaced calls (`pkg::fun()`) are still untracked regardless of `imports` (§1's
   limitation applies independently).
2. `imports` only covers R objects/code, not compiled code.
3. With multiple packages in `imports`, earlier packages' objects override later ones on
   name conflicts.
4. Objects already in `tar_option_get("envir")` override everything in `imports`.

**If your pipeline's logic lives in an R package loaded via `library()`/`load_all()`** rather
than sourced scripts, and `tar_option_set(imports = ...)` is not set, editing that package's
functions will **not** invalidate targets that call them — `tar_make()` will report them as
up to date even though the underlying logic changed. Check this explicitly if your project
has this shape; it's an easy, silent gap.

## 7. `iteration` modes for dynamic branching

- `"vector"` — branching via `vctrs::vec_slice()`, aggregation via `vctrs::vec_c()`.
- `"list"` — branching via `[[]]`, aggregation via `list()`. Appropriate when each branch
  returns a heterogeneous structure (e.g. a named list of several fields) rather than an
  atomic vector.
- `"group"` — `dplyr::group_by()`-like branching over subsets of a static (non-dynamic) data
  frame, using a `tar_group` integer column built by `tar_group()`.

## 8. `tar_outdated()` scope and limits

- `branches = FALSE` (default) reports pattern-level names, not individual branch names, and
  even with `branches = TRUE`, branch names are omitted when they can't be reliably
  determined (e.g. an upstream pattern is itself outdated, so branch identities may shift).
- `shortcut = TRUE` only checks the named targets and trusts stored metadata for everything
  upstream — explicitly documented as "use with caution," since it assumes all upstream
  dependencies are current.
- `tar_outdated()`/`tar_meta()`/`tar_progress()` carry the same "don't call from inside a
  running target" caveat as `tar_read()` (§2).

## 9. Common gotchas worth checking in any new pipeline

- A `pkg::fun()` call anywhere a dependency needs to be tracked (§1).
- Any `tar_read_raw()`/`tar_meta()`/`tar_outdated()` call inside a target's own command body
  (§2) — these should be eliminated via the file-based bridge (§3) or by restructuring to
  read at the orchestration layer, not inside a running target.
- Whether `tar_option_set(imports = ...)` is set if pipeline logic lives in a package rather
  than sourced files (§6) — if not, package-function edits silently fail to invalidate.
- Several configuration profiles writing one `_targets/` store. `tar_read()`
  returns whatever profile was made last, with no warning. Either give each
  profile its own store (`tar_config_set(store = ..., project = ...)`) or
  re-make the profile you want immediately before reading, and record the
  profile in the values object so a report states which one it came from.
