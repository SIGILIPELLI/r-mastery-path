# 07 · Reading Data

## Base R: `read.csv()`

R ships with a CSV reader in base R — no extra packages required:

```r
# sample.csv
# name,age,city
# Alice,23,Boston
# Bob,25,Chicago
# Carol,22,Seattle

people <- read.csv("sample.csv")
people
#     name age    city
# 1  Alice  23  Boston
# 2    Bob  25 Chicago
# 3  Carol  22 Seattle

str(people)
# 'data.frame':	3 obs. of  3 variables:
#  $ name: chr  "Alice" "Bob" "Carol"
#  $ age : int  23 25 22
#  $ city: chr  "Boston" "Chicago" "Seattle"
```

`read.csv()` handles the common case well: it assumes a header row, comma
separators, and infers each column's type automatically.

## Useful `read.csv()` arguments

```r
read.csv("sample.csv", header = TRUE)          # header = TRUE is the default
read.csv("no_header.csv", header = FALSE)      # no header row in the file
read.csv("sample.csv", stringsAsFactors = FALSE)  # character stays character (default since R 4.0)
read.csv("euro.csv", sep = ";")                # semicolon-separated file
read.csv("sample.csv", na.strings = c("", "NA", "N/A"))  # custom missing-value markers
```

!!! info "`stringsAsFactors`"
    Before R 4.0, `read.csv()` converted character columns to `factor`
    (categorical) by default, which surprised a lot of people. Since R 4.0,
    the default is `stringsAsFactors = FALSE`, so text columns load as plain
    `character` unless you ask for factors — the behavior you'll see if
    you're running a current R version.

## The `readr` package: `read_csv()`

The **readr** package (part of the tidyverse) provides a faster, more
predictable alternative with clearer type reporting:

```r
install.packages("readr")   # one-time, see Module 9
library(readr)

people <- read_csv("sample.csv")
# Rows: 3 Columns: 3
# -- Column specification --------
# Delimiter: ","
# chr (2): name, city
# dbl (1): age
#
# i Use `spec()` to retrieve the full column specification for this data.
# i Specify the column types or set `show_col_types = FALSE` to quiet this message.

people
# # A tibble: 3 x 3
#   name    age city
#   <chr> <dbl> <chr>
# 1 Alice    23 Boston
# 2 Bob      25 Chicago
# 3 Carol    22 Seattle
```

`read_csv()` returns a **tibble** — a modern variant of a data frame with
nicer printing (shows types under each column header, truncates long output)
and slightly stricter behavior. Everything you learned about data frames in
[Module 6](06-data-frames-basics.md) still applies; a tibble is a data frame
under the hood.

## `read.csv()` vs. `read_csv()`

| | `read.csv()` (base R) | `read_csv()` (readr) |
|---|---|---|
| Requires a package? | No | Yes (`readr`) |
| Return type | `data.frame` | `tibble` (also a `data.frame`) |
| Speed on large files | Slower | Faster |
| String handling | Character by default (R 4.0+) | Character by default |
| Column type report | Silent | Printed automatically |
| Missing-value markers | `na.strings = c("NA")` | `na = c("NA")` |

For Level 1, either is fine; `readr` becomes the default choice once you
adopt more of the tidyverse in [Level 2](../level-2/01-dplyr-deep-dive.md).

## Writing data back out

```r
write.csv(people, "output.csv", row.names = FALSE)
# row.names = FALSE avoids an extra unnamed index column in the file

# readr equivalent
readr::write_csv(people, "output.csv")   # never adds a row-names column
```

## Reading from a URL

Both functions can read directly from a URL, which is handy for public
datasets without a manual download step:

```r
url <- "https://raw.githubusercontent.com/someuser/somerepo/main/data.csv"
remote_data <- read.csv(url)
```

## Inspecting what you just loaded

Whichever function you use, the same checks are worth running immediately
after loading any new file:

```r
dim(people)       # rows, columns
str(people)       # column types
head(people)      # first 6 rows
summary(people)   # quick stats per column
anyNA(people)     # TRUE if the data has any missing values at all
```

## How It Actually Works

`read.csv()` works by opening a file connection and running R's C-level
tokenizer over it line by line: it splits each line on the separator,
buffers the raw strings for each column, and only *after* seeing every row
does it try to infer each column's type (numeric, integer, logical,
character) by attempting to parse the accumulated strings — which is why
a single stray non-numeric value anywhere in a column forces the whole
column to fall back to `character`. This two-pass-ish behavior (collect
then coerce) is also why huge base-R `read.csv()` calls are memory-hungry:
the raw string buffer and the final typed columns briefly coexist.

`readr::read_csv()` takes a different, faster approach: it reads a sample
of the first ~1000 rows to *guess* column types up front (hence the column
specification it prints), then streams the rest of the file parsing
directly into the guessed types using compiled C++ (via Rcpp) rather than
R-level loops — no full re-scan-and-coerce pass, and no factor conversion
step to worry about. That's the mechanical reason it's both faster and
more predictable on large files.

## Exercise

Create a small CSV file `scores.csv` by hand (a text editor or `cat()` from
R) with columns `student,subject,score` and at least 6 rows across 2
subjects. Load it with `read.csv()`, print `str()` of the result, filter to
rows where `score >= 80`, and write the filtered result to a new file
`passing.csv` with `write.csv(..., row.names = FALSE)`.
