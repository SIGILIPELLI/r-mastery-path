---
title: "Final Quiz"
description: "Test what you have learned across all four levels of R Mastery Path with a 22-question final quiz."
---

# Final Quiz

22 questions drawn from all four levels of this course. You need **70% (16 out of 22)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "What does this print?\n```r\nx <- c(5, NA, 3)\ncat(sum(x), sum(x, na.rm = TRUE), mean(x, na.rm = TRUE), is.na(x[2]), \"\\n\")\n```\n",
    "options": [
      "NA 8 4 FALSE",
      "NA 8 4 TRUE",
      "Error",
      "8 8 4 TRUE"
    ],
    "correctIndex": 1,
    "explanation": "NA propagates unless na.rm = TRUE. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```r\nx <- c(a = 1, b = 2, c = 3)\ncat(x[-1], x[c(\"a\", \"c\")], x[x > 1], length(x[0]), \"\\n\")\n```\n",
    "options": [
      "2 3 1 3 2 3 0",
      "2 3 1 3 2 3 1",
      "Error",
      "1 2 1 3 2 3 0"
    ],
    "correctIndex": 0,
    "explanation": "Negative, named and logical indexing. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```r\nf <- function(n) { total <- 0; for (i in seq_len(n)) total <- total + i; total }\ncat(f(0), f(4), \"\\n\")\n```\n",
    "options": [
      "Error 10",
      "0 4",
      "1 10",
      "0 10"
    ],
    "correctIndex": 3,
    "explanation": "seq_len(0) loops zero times. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```r\nsuppressPackageStartupMessages(library(dplyr))\nd <- data.frame(g = c(\"a\", \"a\", \"b\"), v = c(1, 2, 10))\nr <- d %>% group_by(g) %>% summarise(m = mean(v), n = n(), .groups = \"drop\")\nprint(as.data.frame(r))\n```\n",
    "options": [
      "g m n\n1 a 1.5 2\n2 b 10 1",
      "g    m n\n1 a  1.5 3\n2 b 10.0 1",
      "Error",
      "g    m n\n1 a  1.5 2\n2 b 10.0 1"
    ],
    "correctIndex": 3,
    "explanation": "Group summaries. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```r\nx <- c(10, 20, 30)\nnames(x) <- c(\"a\", \"b\", \"c\")\nlst <- list(x = x, n = 3)\ncat(lst$x[[\"b\"]], lst[[\"n\"]], class(lst[\"n\"]), \"\\n\")\n```\n",
    "options": [
      "Error",
      "20 3 numeric",
      "20 3 list",
      "b 3 list"
    ],
    "correctIndex": 2,
    "explanation": "[ keeps a list; [[ extracts. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```r\ns <- c(\"apple pie\", \"banana\", \"cherry tart\")\ncat(sapply(strsplit(s, \" \"), length), toupper(substr(s, 1, 1)), grepl(\"an\", s), \"\\n\")\n```\n",
    "options": [
      "1 1 1 A B C FALSE TRUE FALSE",
      "2 1 2 A B C FALSE TRUE FALSE",
      "2 1 2 a b c FALSE TRUE FALSE",
      "Error"
    ],
    "correctIndex": 1,
    "explanation": "strsplit, substr and grepl. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```r\nset.seed(7)\nx <- sample(1:10, 5)\nset.seed(7)\ny <- sample(1:10, 5)\ncat(identical(x, y), length(unique(x)), \"\\n\")\n```\n",
    "options": [
      "TRUE 5",
      "Error",
      "FALSE 5",
      "TRUE 10"
    ],
    "correctIndex": 0,
    "explanation": "set.seed makes sampling reproducible. Verified by compiling and running it."
  },
  {
    "question": "What does this print?\n```r\nf <- function(x, ...) UseMethod(\"f\")\nf.default <- function(x, ...) \"default\"\nf.numeric <- function(x, ...) \"number\"\nf.character <- function(x, ...) \"text\"\ncat(f(1), f(\"a\"), f(TRUE), \"\\n\")\n```\n",
    "options": [
      "number text number",
      "default default default",
      "number text default",
      "Error"
    ],
    "correctIndex": 2,
    "explanation": "S3 dispatch on class. Verified by compiling and running it."
  },
  {
    "question": "A script works on your laptop but not on a colleague's. What helps most?",
    "options": [
      "Ignore",
      "Email",
      "Pin package versions with renv and document the R version",
      "Reinstall"
    ],
    "correctIndex": 2,
    "explanation": "Reproducibility."
  },
  {
    "question": "A for loop over 1:length(x) fails when x is empty. What's the fix?",
    "options": [
      "Add if",
      "Sort",
      "Use while",
      "Use seq_along(x) or seq_len(length(x))"
    ],
    "correctIndex": 3,
    "explanation": "1:0 is c(1, 0)."
  },
  {
    "question": "A join produces more rows than expected. What do you check?",
    "options": [
      "Column order",
      "Types",
      "Duplicate keys on either side",
      "Names"
    ],
    "correctIndex": 2,
    "explanation": "Many-to-many joins multiply rows."
  },
  {
    "question": "What is the safest way to read a CSV where IDs have leading zeros?",
    "options": [
      "Default",
      "Numeric",
      "Factor",
      "Read the column as character (colClasses or col_types)"
    ],
    "correctIndex": 3,
    "explanation": "Avoid losing zeros."
  },
  {
    "question": "When is a loop fine in R?",
    "options": [
      "Always",
      "Never",
      "Rarely",
      "When logic depends on previous iterations or clarity matters more than speed"
    ],
    "correctIndex": 3,
    "explanation": "Preallocate."
  },
  {
    "question": "What does expect_equal use for numbers?",
    "options": [
      "Exact",
      "A tolerance, so tiny floating point differences pass",
      "Strings",
      "Types"
    ],
    "correctIndex": 1,
    "explanation": "Use expect_identical for exact."
  },
  {
    "question": "Why write R functions with stopifnot or clear errors?",
    "options": [
      "Speed",
      "Fail early with understandable messages when inputs are invalid",
      "Law",
      "Style"
    ],
    "correctIndex": 1,
    "explanation": "Defensive code."
  },
  {
    "question": "What belongs in tests/testthat?",
    "options": [
      "Plots",
      "Data",
      "Automated tests named test-*.R using expect_* functions",
      "Docs"
    ],
    "correctIndex": 2,
    "explanation": "Package testing."
  },
  {
    "question": "What makes a Shiny app slow?",
    "options": [
      "UI",
      "Names",
      "Colors",
      "Recomputing heavy work on every reactive update"
    ],
    "correctIndex": 3,
    "explanation": "Cache and precompute."
  },
  {
    "question": "What does an R Markdown report give you?",
    "options": [
      "Data",
      "Package",
      "App",
      "Narrative plus code and results in one reproducible document"
    ],
    "correctIndex": 3,
    "explanation": "knitr."
  },
  {
    "question": "What do you do before trusting a regression?",
    "options": [
      "Plot",
      "Check residuals, outliers and assumptions, not just p-values",
      "R²",
      "Nothing"
    ],
    "correctIndex": 1,
    "explanation": "Diagnostics."
  },
  {
    "question": "A p-value is 0.04 with n = 20000. What matters too?",
    "options": [
      "Effect size and practical significance",
      "Name",
      "Nothing",
      "Only p"
    ],
    "correctIndex": 0,
    "explanation": "Big samples find tiny effects."
  },
  {
    "question": "How do you speed up grouped aggregation on 50M rows?",
    "options": [
      "Loop",
      "Base",
      "Use data.table (or arrow/duckdb) instead of copying data repeatedly",
      "Print"
    ],
    "correctIndex": 2,
    "explanation": "Efficient engines."
  },
  {
    "question": "What does plumber add to R code?",
    "options": [
      "An HTTP API around your functions",
      "Docs",
      "Tests",
      "Plots"
    ],
    "correctIndex": 0,
    "explanation": "Deployment."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### Clean, validate and summarise a messy customer file

A raw customer table has duplicates, messy names and emails, mixed date formats and bad numbers. Standardise it with dplyr/stringr/lubridate, flag each bad row with a reason (case_when), split good from rejected, and report counts, reject reasons, the good rows, monthly revenue, the total and the days since the first signup.

**Starter code**

```r
suppressPackageStartupMessages({ library(dplyr); library(stringr); library(lubridate); library(tidyr) })

raw <- data.frame(
  id    = c(1, 2, 2, 3, 4, 5, 6),
  name  = c(" Ann Lee ", "bob  stone", "bob  stone", "CY ray", NA, "Dee Fox", "eli  park"),
  email = c("ann@x.com", "BOB@X.COM", "BOB@X.COM", "cy-at-x.com", "dee@x.com", "dee2@x.com", "eli@x.com"),
  signup = c("2026-01-05", "2026/02/10", "2026/02/10", "2026-03-01", "not a date", "2026-04-15", "2026-04-20"),
  spend = c("120.50", "75", "75", "abc", "40", "-5", "300"),
  stringsAsFactors = FALSE
)

# TODO 1: remove duplicate rows (distinct)
# TODO 2: standardise: name -> squish whitespace + title case; email -> lower case; signup -> Date (formats like 2026-01-05 or 2026/02/10);
#         spend -> numeric (non-numeric becomes NA without printing a warning)
# TODO 3: add a `problem` column using case_when in this order:
#         "missing name", "bad email" (must look like x@y.z), "bad date", "bad spend", "negative spend", otherwise NA
# TODO 4: good = rows without a problem (signup as a Date); bad = rows with one
# TODO 5: print:  raw rows / after distinct / good / rejected, the reject reasons as id=reason,
#         the good rows (id, name, email, signup, spend), monthly customers and revenue, total revenue,
#         and days from the first signup to 2026-05-01
```

**Expected output**

```text
raw rows: 7 | after distinct: 6 | good: 3 | rejected: 3 
reject reasons: 3=bad email, 4=missing name, 5=negative spend 
  id      name     email     signup spend
1  1   Ann Lee ann@x.com 2026-01-05 120.5
2  2 Bob Stone bob@x.com 2026-02-10  75.0
3  6  Eli Park eli@x.com 2026-04-20 300.0
    month customers revenue
1 2026-01         1   120.5
2 2026-02         1    75.0
3 2026-04         1   300.0
total revenue: 495.5 
days since first signup (to 2026-05-01): 116
```

<details><summary>Show solution and explanation</summary>

<pre><code>suppressPackageStartupMessages({ library(dplyr); library(stringr); library(lubridate); library(tidyr) })

raw &lt;- data.frame(
  id    = c(1, 2, 2, 3, 4, 5, 6),
  name  = c(&quot; Ann Lee &quot;, &quot;bob  stone&quot;, &quot;bob  stone&quot;, &quot;CY ray&quot;, NA, &quot;Dee Fox&quot;, &quot;eli  park&quot;),
  email = c(&quot;ann@x.com&quot;, &quot;BOB@X.COM&quot;, &quot;BOB@X.COM&quot;, &quot;cy-at-x.com&quot;, &quot;dee@x.com&quot;, &quot;dee2@x.com&quot;, &quot;eli@x.com&quot;),
  signup = c(&quot;2026-01-05&quot;, &quot;2026/02/10&quot;, &quot;2026/02/10&quot;, &quot;2026-03-01&quot;, &quot;not a date&quot;, &quot;2026-04-15&quot;, &quot;2026-04-20&quot;),
  spend = c(&quot;120.50&quot;, &quot;75&quot;, &quot;75&quot;, &quot;abc&quot;, &quot;40&quot;, &quot;-5&quot;, &quot;300&quot;),
  stringsAsFactors = FALSE
)

clean &lt;- raw %&gt;%
  distinct() %&gt;%
  mutate(
    name   = str_to_title(str_squish(name)),
    email  = str_to_lower(email),
    signup = parse_date_time(signup, orders = c(&quot;Ymd&quot;)),
    spend  = suppressWarnings(as.numeric(spend)),
    problem = case_when(
      is.na(name) ~ &quot;missing name&quot;,
      !str_detect(email, &quot;^[^@\\s]+@[^@\\s]+\\.[a-z]+$&quot;) ~ &quot;bad email&quot;,
      is.na(signup) ~ &quot;bad date&quot;,
      is.na(spend) ~ &quot;bad spend&quot;,
      spend &lt; 0 ~ &quot;negative spend&quot;,
      TRUE ~ NA_character_
    )
  )

good &lt;- clean %&gt;% filter(is.na(problem)) %&gt;% mutate(signup = as_date(signup))
bad  &lt;- clean %&gt;% filter(!is.na(problem))

cat(&quot;raw rows:&quot;, nrow(raw), &quot;| after distinct:&quot;, nrow(distinct(raw)), &quot;| good:&quot;, nrow(good), &quot;| rejected:&quot;, nrow(bad), &quot;\n&quot;)
cat(&quot;reject reasons:&quot;, paste(bad$id, bad$problem, sep = &quot;=&quot;, collapse = &quot;, &quot;), &quot;\n&quot;)
print(as.data.frame(good %&gt;% select(id, name, email, signup, spend)))

monthly &lt;- good %&gt;% mutate(month = format(signup, &quot;%Y-%m&quot;)) %&gt;%
  group_by(month) %&gt;% summarise(customers = n(), revenue = sum(spend), .groups = &quot;drop&quot;)
print(as.data.frame(monthly))
cat(&quot;total revenue:&quot;, sum(good$spend), &quot;\n&quot;)
cat(&quot;days since first signup (to 2026-05-01):&quot;, as.integer(as_date(&quot;2026-05-01&quot;) - min(good$signup)), &quot;\n&quot;)
</code></pre>

<p>A tidy pipeline standardises first, then validates with ordered rules so each bad row gets exactly one reason, and nothing is silently dropped. Verified by running the script with R 4.6 and the tidyverse packages.</p>
</details>

### Write and test functions: moving average, vectorised running max and a bootstrap interval

Implement moving_avg (validated, vapply), a loop and a vectorised running maximum, and a reproducible bootstrap confidence interval. The script's check() helper runs testthat expectations and prints PASS/FAIL; one expectation is deliberately wrong to show a failure.

**Starter code**

```r
suppressPackageStartupMessages(library(testthat))

# ---- code under test ----
# TODO: implement
#  moving_avg(x, k): validate (numeric x, whole number k >= 1 -> stopifnot), return numeric(0) when k > length(x),
#                    otherwise the mean of every window of k consecutive values (use vapply)
#  cum_max_loop(x): running maximum using a loop; cum_max_vec(x): the vectorised version
#  bootstrap_ci(x, stat = mean, reps = 400, level = 0.95, seed = 1): set.seed(seed), resample with replacement `reps` times,
#                    return the unnamed lower and upper quantiles
moving_avg <- function(x, k) NULL
cum_max_loop <- function(x) NULL
cum_max_vec <- function(x) NULL
bootstrap_ci <- function(x, stat = mean, reps = 400, level = 0.95, seed = 1) NULL

# ---- checks (each prints PASS/FAIL instead of stopping) ----
check <- function(name, expr) {
  ok <- tryCatch({ force(expr); TRUE }, error = function(e) FALSE)
  cat(sprintf("%-34s %s\n", name, if (ok) "PASS" else "FAIL"))
  invisible(ok)
}
x <- c(3, 1, 4, 1, 5, 9, 2, 6)
check("moving_avg basic", expect_equal(moving_avg(x, 3), c(8/3, 2, 10/3, 5, 16/3, 17/3)))
check("moving_avg window = length", expect_equal(moving_avg(x, 8), mean(x)))
check("moving_avg window too large", expect_equal(moving_avg(x, 9), numeric(0)))
check("moving_avg rejects bad k", expect_error(moving_avg(x, 0)))
check("moving_avg rejects text", expect_error(moving_avg("a", 2)))
check("loop and vectorised agree", expect_identical(cum_max_loop(x), cum_max_vec(x)))
ci <- bootstrap_ci(x)
check("bootstrap interval ordered", expect_lt(ci[1], ci[2]))
check("bootstrap reproducible", expect_equal(bootstrap_ci(x), bootstrap_ci(x)))
check("bootstrap contains the mean", expect_true(ci[1] <= mean(x) && mean(x) <= ci[2]))
check("deliberately wrong expectation", expect_equal(moving_avg(x, 2)[1], 99))

cat("moving_avg(x, 3):", round(moving_avg(x, 3), 2), "\n")
cat("cummax:", cum_max_vec(x), "\n")
cat("95% bootstrap CI for the mean:", round(ci, 2), "\n")
big <- as.numeric(1:200000)
cat("same answer on 200k values:", identical(cum_max_loop(big), cum_max_vec(big)), "\n")
```

**Expected output**

```text
moving_avg basic                   PASS
moving_avg window = length         PASS
moving_avg window too large        PASS
moving_avg rejects bad k           PASS
moving_avg rejects text            PASS
loop and vectorised agree          PASS
bootstrap interval ordered         PASS
bootstrap reproducible             PASS
bootstrap contains the mean        PASS
deliberately wrong expectation     FAIL
moving_avg(x, 3): 2.67 2 3.33 5 5.33 5.67 
cummax: 3 3 4 4 5 9 9 9 
95% bootstrap CI for the mean: 2.12 6 
same answer on 200k values: TRUE
```

<details><summary>Show solution and explanation</summary>

<pre><code>suppressPackageStartupMessages(library(testthat))

# ---- code under test ----
moving_avg &lt;- function(x, k) {
  stopifnot(is.numeric(x), k &gt;= 1, k == round(k))
  if (k &gt; length(x)) return(numeric(0))
  vapply(seq_len(length(x) - k + 1), function(i) mean(x[i:(i + k - 1)]), numeric(1))
}
cum_max_loop &lt;- function(x) { out &lt;- numeric(length(x)); m &lt;- -Inf; for (i in seq_along(x)) { m &lt;- max(m, x[i]); out[i] &lt;- m }; out }
cum_max_vec &lt;- function(x) cummax(x)

bootstrap_ci &lt;- function(x, stat = mean, reps = 400, level = 0.95, seed = 1) {
  set.seed(seed)
  b &lt;- replicate(reps, stat(sample(x, replace = TRUE)))
  unname(quantile(b, c((1 - level) / 2, 1 - (1 - level) / 2)))
}

# ---- checks (each prints PASS/FAIL instead of stopping) ----
check &lt;- function(name, expr) {
  ok &lt;- tryCatch({ force(expr); TRUE }, error = function(e) FALSE)
  cat(sprintf(&quot;%-34s %s\n&quot;, name, if (ok) &quot;PASS&quot; else &quot;FAIL&quot;))
  invisible(ok)
}
x &lt;- c(3, 1, 4, 1, 5, 9, 2, 6)
check(&quot;moving_avg basic&quot;, expect_equal(moving_avg(x, 3), c(8/3, 2, 10/3, 5, 16/3, 17/3)))
check(&quot;moving_avg window = length&quot;, expect_equal(moving_avg(x, 8), mean(x)))
check(&quot;moving_avg window too large&quot;, expect_equal(moving_avg(x, 9), numeric(0)))
check(&quot;moving_avg rejects bad k&quot;, expect_error(moving_avg(x, 0)))
check(&quot;moving_avg rejects text&quot;, expect_error(moving_avg(&quot;a&quot;, 2)))
check(&quot;loop and vectorised agree&quot;, expect_identical(cum_max_loop(x), cum_max_vec(x)))
ci &lt;- bootstrap_ci(x)
check(&quot;bootstrap interval ordered&quot;, expect_lt(ci[1], ci[2]))
check(&quot;bootstrap reproducible&quot;, expect_equal(bootstrap_ci(x), bootstrap_ci(x)))
check(&quot;bootstrap contains the mean&quot;, expect_true(ci[1] &lt;= mean(x) &amp;&amp; mean(x) &lt;= ci[2]))
check(&quot;deliberately wrong expectation&quot;, expect_equal(moving_avg(x, 2)[1], 99))

cat(&quot;moving_avg(x, 3):&quot;, round(moving_avg(x, 3), 2), &quot;\n&quot;)
cat(&quot;cummax:&quot;, cum_max_vec(x), &quot;\n&quot;)
cat(&quot;95% bootstrap CI for the mean:&quot;, round(ci, 2), &quot;\n&quot;)
big &lt;- as.numeric(1:200000)
cat(&quot;same answer on 200k values:&quot;, identical(cum_max_loop(big), cum_max_vec(big)), &quot;\n&quot;)
</code></pre>

<p>Validation with stopifnot fails early, vapply guarantees the result type, vectorised code gives the same answer as a loop but faster, and a fixed seed makes bootstrapping reproducible. Verified by running the script with R 4.6 and testthat.</p>
</details>
