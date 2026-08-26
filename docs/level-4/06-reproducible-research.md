# 06 · Reproducible Research Practices

"It works on my machine" is an especially expensive failure in
research: a result nobody else can regenerate is a result nobody can
trust. `renv` solves the package half of that problem — pinning exact
package versions to a project instead of your global library — and a
consistent project layout plus version control solves the rest.

```r
library(renv)
```

## Initializing a project library

`renv::init()` creates a private package library scoped to the current
project, plus an `.Rprofile` that activates it automatically every time
the project is opened:

```r
renv::init(bare = TRUE)
```

```
- The project is out-of-sync -- use `renv::status()` for details.
```

That message isn't an error — it's `renv` telling you the project now
has its own library, but the packages your scripts `library()` haven't
been snapshotted into a lockfile yet. `bare = TRUE` skips scanning for
and installing existing dependencies immediately, which is useful when
you want to add them incrementally.

After `init()`, the project directory has:

```
myproject/
  .Rprofile        # auto-activates renv for this project
  renv/
    activate.R     # the activation script .Rprofile sources
    library/       # project-private package installs live here
  renv.lock        # not yet written until the first snapshot
```

## Recording exact versions with a lockfile

`renv::snapshot()` scans your R files for `library()`/`require()` calls
and records the exact installed version of each into `renv.lock`:

```r
renv::snapshot(prompt = FALSE)
```

```
- The project is out-of-sync -- use `renv::status()` for details.
The following required packages are not installed:
- jsonlite
Packages must first be installed before renv can snapshot them.
Use `renv::dependencies()` to see where this package is used in your project.

The following package(s) will be updated in the lockfile:

# CRAN -----------------------------------------------------------------------
- renv   [* -> 1.2.4]

The version of R recorded in the lockfile will be updated:
- R      [* -> 4.6.1]

- Lockfile written to ".../renv.lock".
```

This run is instructive precisely because it's incomplete: the project
script called `library(jsonlite)`, but `jsonlite` was never installed
into the project's private library — so `snapshot()` records only what
*is* installed (`renv` itself, R's own version) and warns rather than
silently pretending the missing package doesn't matter. Checking
`renv::status()` afterward confirms the same gap:

```r
renv::status()
```

```
- The project is out-of-sync -- use `renv::status()` for details.
The following package(s) are used in this project, but are not installed:
- jsonlite

See `?renv::status` for advice on resolving these issues.
```

The fix is always `renv::install("jsonlite")` followed by another
`renv::snapshot()` — install first, then record.

`renv.lock` itself is JSON and is meant to be committed to version
control:

```json
{
  "R": {
    "Version": "4.6.1",
    "Repositories": [{ "Name": "CRAN", "URL": "https://cloud.r-project.org" }]
  },
  "Packages": {
    "renv": {
      "Package": "renv",
      "Version": "1.2.4",
      "Source": "Repository"
    }
  }
}
```

## Restoring a project on a different machine

Anyone who clones the repo and opens it in R gets the exact same package
versions with one call, which installs into their own project-local
library without touching their global one:

```r
renv::restore()
```

This is the payoff: the lockfile plus `restore()` replaces "please
install these packages, roughly these versions, good luck" with a
single reproducible command.

## A reproducible project layout

Beyond package versions, a project that another person (or a CI runner)
can rerun end to end tends to share this shape:

```
myproject/
  renv.lock          # exact package versions — committed
  .Rprofile          # activates renv — committed
  data/
    raw/             # original data, never edited in place
    processed/       # output of scripts, regenerable, often gitignored
  R/                 # reusable functions
  analysis/
    01_clean.R
    02_model.R
    03_report.Rmd
  .gitignore         # excludes renv/library/, data/processed/, etc.
```

The convention worth internalizing: raw data is read-only and
regenerable outputs are never hand-edited — if `02_model.R` produces a
`.rds` file, that file is disposable, and rerunning the numbered scripts
in order should always reproduce it bit-for-bit given the same
`renv.lock`.

## R-specific traps

**`renv::init()` rewrites `.Rprofile`**, and if a project already has a
custom `.Rprofile` for something else, `init()` appends to it rather
than replacing it — check the file after running `init()` on an
existing project rather than assuming it started from nothing.

**A lockfile records package versions, not R's own base packages or
system dependencies** (a compiler for a package with C++ code, a system
library like `libcurl`) — `renv::restore()` reproduces the R package
graph, not the OS. Document those separately (a Dockerfile, a README
section) if the project depends on them.

**`renv`'s private library is per-project, not per-user** — running
`renv::restore()` on a laptop with limited disk and dozens of `renv`
projects means dozens of separate copies of common packages like
`ggplot2`, since renv does not share libraries across projects by
default (it does cache downloads globally, so re-installs are fast, but
each project's `renv/library/` is its own full copy).

## Cheat sheet

| Task | Call |
|---|---|
| Create a project-local library | `renv::init()` |
| Create one without auto-installing existing deps | `renv::init(bare = TRUE)` |
| Record exact package versions | `renv::snapshot()` |
| Check for drift between code and lockfile | `renv::status()` |
| Reinstall exactly what the lockfile records | `renv::restore()` |
| Install a package into the project library | `renv::install("pkgname")` |
| See where a package is used in the project | `renv::dependencies()` |

## Exercise

1. Create a new project directory, run `renv::init(bare = TRUE)`, write
   a script that calls `library(jsonlite)`, install it with
   `renv::install("jsonlite")`, then run `renv::snapshot()` and inspect
   the resulting `renv.lock`.
2. Delete the project's `renv/library` directory (simulating a fresh
   clone with no packages installed) and run `renv::restore()` to
   confirm it reinstalls exactly what the lockfile specifies.
3. Sketch a `.gitignore` for the project layout above that keeps
   `renv.lock` and `.Rprofile` tracked but excludes `renv/library/` and
   `data/processed/`.
