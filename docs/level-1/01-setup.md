# 01 · Setup & First Script

## Install R

R is the language and runtime; **RStudio** is the most popular IDE for working
with it. Install both.

```bash
# macOS (Homebrew)
brew install r
brew install --cask rstudio

# Ubuntu/Debian
sudo apt update
sudo apt install r-base

# Windows: download installers from https://cran.r-project.org
# and https://posit.co/download/rstudio-desktop/
```

Verify the install:

```bash
R --version
# R version 4.x.x (2024-xx-xx) -- "..."

Rscript --version
# Rscript (R) version 4.x.x
```

`R` starts an interactive console (the **REPL**); `Rscript` runs a `.R` file
from the command line without opening the console.

## Running your first script

Create `hello.R`:

```r
# hello.R
message <- "Hello, world!"
print(message)
```

Run it from the command line:

```bash
Rscript hello.R
# [1] "Hello, world!"
```

The `[1]` prefix is R's way of labeling the first element of the printed
vector — you'll see this constantly, since almost everything in R is a
vector, even a single string.

## The interactive console (REPL)

Type `R` in a terminal to get an interactive prompt. This is where most R
users experiment before saving code to a script:

```r
> 2 + 2
[1] 4
> x <- 10
> x * 3
[1] 30
> quit()
```

`quit()` (or `q()`) exits the console. When it asks "Save workspace image?",
answer `n` while learning — you generally want scripts, not saved sessions, to
be your source of truth.

## `print()` vs. auto-printing

At the top level of the console (or when sourced with `source()`), a bare
expression auto-prints its value. Inside a script run with `Rscript`, or
inside a function, nothing auto-prints — you need an explicit `print()`.

```r
# In an interactive console, this alone would print [1] 42:
42

# In a script run with Rscript, you must be explicit:
print(42)
# [1] 42
```

`cat()` is a lighter-weight alternative to `print()` for plain output —  it
doesn't add the `[1]` index prefix or quote strings:

```r
cat("Hello, world!\n")
# Hello, world!

print("Hello, world!")
# [1] "Hello, world!"
```

## Comments and basic syntax

```r
# This is a comment -- R has no multi-line comment syntax,
# so consecutive lines each need their own #

x <- 5        # assignment with the arrow operator
y = 10        # also valid, but "<-" is the idiomatic convention
x + y         # 15 -- arithmetic works as expected
```

R is **case-sensitive** (`x` and `X` are different variables), statements
don't require a trailing semicolon (though `;` can separate multiple
statements on one line), and whitespace/indentation is stylistic, not
syntactic — unlike Python.

## Working directory and RStudio projects

R reads and writes files relative to its **working directory**. Check and set
it with:

```r
getwd()
# [1] "/Users/you/some/path"

setwd("/Users/you/projects/r-lessons")
```

In practice, most R users avoid `setwd()` in scripts (it hardcodes a path
that won't exist on another machine) and instead open an **RStudio Project**
(`File > New Project`), which sets the working directory to the project
folder automatically and keeps related scripts and data together.

## Anatomy of a script run

| Piece | Meaning |
|-------|---------|
| `Rscript file.R` | Runs `file.R` from the command line, non-interactively. |
| `R` | Opens the interactive console (REPL). |
| `<-` | The assignment operator (idiomatic; `=` also works at the top level). |
| `#` | Starts a comment; runs to the end of the line. |
| `print(x)` / `cat(x)` | Explicitly display a value's contents. |
| `getwd()` / `setwd()` | Get/set the working directory for file paths. |

## Choosing an editor

**RStudio** is the standard choice for R work — it has an integrated console,
a variable/environment viewer, an built-in plot pane, and one-click package
installation, all of which matter a lot for data analysis. **VS Code** with
the "R" extension is a lighter-weight alternative if you already live in VS
Code for other languages. Either is fine to start; RStudio is what almost all
R tutorials and documentation assume, so it's the path of least friction.

## Exercise

Write a script `greet.R` that creates a variable holding your name, then
prints a greeting that includes it using both `cat()` and `print()` so you can
see the difference in their output formatting. Run it with `Rscript greet.R`.
