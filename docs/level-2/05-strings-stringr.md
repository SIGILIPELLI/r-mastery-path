# 05 · Working with Strings (stringr)

Base R has string functions (`nchar()`, `substr()`, `paste()`, `grepl()`),
but their argument order and naming are famously inconsistent — some take
the pattern first, some the string first, some use `fixed = TRUE` for
literal matching and some use different flags entirely. **stringr** wraps
the same underlying capability (via the `stringi` package) in a
consistent family of functions that all start with `str_` and always take
the string as the first argument, making them easy to use in a `dplyr`
pipe.

```r
library(stringr)

products <- c("  Widget A ", "gadget-X", "TOOL_Z", "Gizmo Q99")
```

## Basic operations

```r
str_length(products)          # character count, like nchar()
# [1] 11  8  6  9

str_trim(products)            # remove leading/trailing whitespace
# [1] "Widget A" "gadget-X" "TOOL_Z"   "Gizmo Q99"

str_to_lower(products)
# [1] "  widget a " "gadget-x"    "tool_z"      "gizmo q99"

str_to_upper(products)
str_to_title(products)        # "  Widget A ", "Gadget-X", "Tool_z", "Gizmo Q99"
```

Every one of these takes the string first — no need to remember whether a
particular base function wants `nchar(x)` or `substr(x, start, stop)` order.

## Detecting and matching patterns

```r
str_detect(products, "gadget")             # case-sensitive by default
# [1] FALSE  TRUE FALSE FALSE

str_detect(str_to_lower(products), "gadget")
# [1] FALSE  TRUE FALSE FALSE

str_starts(products, "  ")   # does it start with this literal text?
str_ends(products, "9")      # does it end with this?

str_count(products, "[aeiouAEIOU]")   # count pattern occurrences per string
# [1] 3 2 2 2
```

`str_detect()` returns one `TRUE`/`FALSE` per element — the stringr
equivalent of base R's `grepl()`, but with the string always first and no
separate `pattern`/`x` ordering to remember.

## Extracting matches

```r
codes <- c("ORD-2024-001", "ORD-2024-002", "INV-2023-099")

str_extract(codes, "[0-9]{4}")          # first match of a 4-digit run
# [1] "2024" "2024" "2023"

str_extract(codes, "^[A-Z]+")           # leading letters
# [1] "ORD" "ORD" "INV"

str_match(codes, "([A-Z]+)-([0-9]{4})-([0-9]+)")
#      [,1]           [,2]  [,3]   [,4]
# [1,] "ORD-2024-001" "ORD" "2024" "001"
# [2,] "ORD-2024-002" "ORD" "2024" "002"
# [3,] "INV-2023-099" "INV" "2023" "099"
```

`str_match()` with parenthesized groups `()` in the pattern returns a
matrix: column 1 is the whole match, and each subsequent column is one
capture group — useful for pulling several pieces out of a structured
string (like a code, an ID, or a log line) in one call.

## Replacing text

```r
str_replace(products, "_", " ")          # replaces the FIRST match only
# [1] "  Widget A "  "gadget-X"     "TOOL Z"       "Gizmo Q99"

str_replace_all(products, "[-_]", " ")   # replaces EVERY match
# [1] "  Widget A "  "gadget X"     "TOOL Z"       "Gizmo Q99"
```

**Trap:** `str_replace()` (singular) only touches the *first* match per
string — a common surprise for people expecting global replacement by
default, since most other languages' basic "replace" defaults to global.
Reach for `str_replace_all()` whenever you mean "every occurrence."

## Splitting and joining

```r
str_split("2024-01-15", "-")
# [[1]]
# [1] "2024" "01"   "15"

str_split_fixed("2024-01-15", "-", n = 3)   # returns a matrix, not a list
#      [,1]   [,2] [,3]
# [1,] "2024" "01" "15"

str_c("Order", "#", 42, sep = "-")           # concatenate, like paste()
# [1] "Order-#-42"

str_c(c("a", "b", "c"), collapse = ", ")     # join a vector into one string
# [1] "a, b, c"
```

`str_split()` returns a **list** (one element per input string, since
different strings can split into different numbers of pieces) — a subtlety
that trips people up when they expect a flat vector back. Use
`str_split_fixed()` when you know every string splits into the same fixed
number of parts and want a matrix/data frame directly, and `unlist()` on
`str_split()`'s output if you only ever had one string to begin with.

## Padding and truncating

```r
str_pad(c("1", "22", "333"), width = 5, pad = "0")
# [1] "00001" "00022" "00333"

str_trunc("This is a very long product description", width = 20)
# [1] "This is a very lo..."
```

`str_pad()` is the idiomatic way to zero-pad IDs or align text columns
without hand-rolling `sprintf("%05d", x)` logic for non-numeric strings.

## Regular expressions vs. fixed strings

Every stringr function that takes a pattern treats it as a **regular
expression** by default. If your "pattern" is a literal string that
happens to contain regex metacharacters (`.`, `+`, `(`, `[`, ...), wrap it
in `fixed()` to match it literally instead:

```r
prices <- c("$4.99", "$12.00", "4.99")

str_detect(prices, "4.99")           # "." matches ANY character here -- true for "4x99" too
str_detect(prices, fixed("4.99"))    # matches the literal text "4.99" only
```

This is a very common silent bug: a pattern like `"3.14"` or a file
extension like `".csv"` will match more than intended because `.` in regex
means "any character," not "a literal period."

## stringr cheat sheet

| Task | Function |
|------|----------|
| Length | `str_length(x)` |
| Trim whitespace | `str_trim(x)` |
| Change case | `str_to_lower(x)`, `str_to_upper(x)`, `str_to_title(x)` |
| Test for a pattern | `str_detect(x, pattern)` |
| Count occurrences | `str_count(x, pattern)` |
| Extract first match | `str_extract(x, pattern)` |
| Extract capture groups | `str_match(x, pattern)` |
| Replace first match | `str_replace(x, pattern, replacement)` |
| Replace all matches | `str_replace_all(x, pattern, replacement)` |
| Split | `str_split(x, pattern)` (list) or `str_split_fixed(x, pattern, n)` (matrix) |
| Concatenate | `str_c(..., sep = "")` |
| Join a vector into one string | `str_c(x, collapse = ", ")` |
| Pad | `str_pad(x, width, pad = "0")` |
| Match literally, not as regex | `str_detect(x, fixed("..."))` |

## Exercise

Given `emails <- c("alice@example.com", "bob.smith@company.org",
"not-an-email")`: use `str_detect()` with a regex pattern to build a
logical vector flagging which entries look like a valid email (contain
`@` and at least one `.` after it), then use `str_extract()` to pull out
just the domain (everything after `@`) for the entries that pass. Finally,
use `fixed()` to demonstrate the difference between matching a literal `.`
in `"company.org"` versus an unescaped `.` in a regex pattern.
