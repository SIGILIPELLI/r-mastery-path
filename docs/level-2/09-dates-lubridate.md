# 09 · Working with Dates (lubridate)

Base R's `Date` and `POSIXct` classes can parse and compute with dates, but
their parsing functions (`as.Date()`, `strptime()`) demand exact format
strings and their arithmetic has sharp edges around time zones and daylight
saving. **lubridate** wraps the same underlying date/time machinery in
functions that are far more forgiving to parse and far clearer to reason
about — it's the tidyverse's answer to "why is date handling always more
annoying than it should be."

```r
library(lubridate)
```

## Parsing dates — order-based helpers

Instead of memorizing `strftime()`-style format codes (`%Y-%m-%d`),
lubridate offers functions named after the order the components appear in:

```r
ymd("2024-03-15")        # Year, Month, Day
# [1] "2024-03-15"
mdy("March 15, 2024")    # Month, Day, Year
# [1] "2024-03-15"
dmy("15/03/2024")        # Day, Month, Year
# [1] "2024-03-15"
ymd_hms("2024-03-15 14:30:00")   # with time
# [1] "2024-03-15 14:30:00 UTC"
```

All four accept a wide range of separators (`-`, `/`, spaces, no separator
at all) automatically — `ymd("20240315")` and `ymd("2024/03/15")` both
parse successfully without any format string.

**Trap:** an ambiguous date like `"03/04/2024"` means March 4th under `mdy()`
but April 3rd under `dmy()` — lubridate correctly parses either
interpretation *if you pick the matching function*, but it cannot detect
which one you meant from the string alone. Always match the parsing
function to the actual source format of your data, and check a few known
dates after parsing when working with an unfamiliar dataset.

## Extracting components

```r
my_date <- ymd("2024-03-15")

year(my_date)      # 2024
month(my_date)      # 3
day(my_date)         # 15
wday(my_date, label = TRUE)   # Fri (day of week, as a label)
yday(my_date)        # 75 -- day of the year
```

## Date arithmetic

```r
my_date + days(10)
# [1] "2024-03-25"
my_date - months(1)
# [1] "2024-02-15"
my_date + years(1)
# [1] "2025-03-15"

ymd("2024-06-01") - ymd("2024-03-15")
# Time difference of 78 days
```

`days()`, `months()`, `years()` create lubridate **periods** — human
calendar units. `months(1)` added to Jan 31 correctly lands on Feb 28/29
(clamped to the shorter month) rather than overflowing into March, matching
how a person would think about "one month later."

## Periods vs. durations — a genuine R-specific trap

lubridate has two different concepts for "an amount of time," and mixing
them up gives subtly wrong answers around daylight saving time boundaries:

```r
# A PERIOD: calendar time -- "one day" always means one calendar day
ymd_hms("2024-03-09 12:00:00", tz = "America/New_York") + days(1)
# Follows the calendar -- lands on March 10, 12:00:00, regardless of DST

# A DURATION: exact elapsed time -- "one day" always means exactly 86400 seconds
ymd_hms("2024-03-09 12:00:00", tz = "America/New_York") + ddays(1)
# Adds exactly 24 hours of elapsed time -- may land on a different clock time
# across a DST transition (e.g. 13:00:00 if clocks sprang forward that night)
```

| Concept | Function prefix | Means |
|---------|-----------------|-------|
| Period | `days()`, `months()`, `years()` | calendar time (follows human calendar rules, DST-aware) |
| Duration | `ddays()`, `dhours()`, `dminutes()` | exact elapsed physical time in seconds |

For "what date is 30 days from now" (calendar thinking), use periods. For
"exactly how much wall-clock time has elapsed" (physical/scientific
thinking), use durations. Picking the wrong one rarely matters for
same-day arithmetic, but silently produces off-by-one-hour bugs across
daylight saving transitions if you're not deliberate about which one you
mean.

## Formatting dates for display

```r
format(my_date, "%B %d, %Y")     # base R's format() still works on lubridate dates
# [1] "March 15, 2024"
```

lubridate also has `stamp()`, which is supposed to let you skip
`%B %d, %Y`-style format codes by giving one example of the output you
want and inferring the pattern to apply to other dates. In practice it's
worth knowing about but not fully trusting:

```r
stamp("March 15, 2024")(my_date)
```

```text
Multiple formats matched: "%Om %d, %y%H"(1), "%Om %y, %d%H"(1),
"%Om %d, %Y"(1), "%B %d, %y%H"(1), "%B %y, %d%H"(1), "%B %d, %Y"(1),
"March %H, %M%S"(1)
Using: "%B %y, %d%H"
[1] "March 24, 1500"
```

**Trap:** because the example date `"March 15, 2024"` has numbers that
could plausibly be a day, a year, an hour, or minutes/seconds, `stamp()`
finds *several* equally plausible interpretations, silently picks one
(often not the one you meant), and only tells you it guessed via a printed
message rather than an error — easy to miss if you're not scanning the
console output. The safe, unambiguous choice is `format()` with an
explicit format string; use `stamp()` only with example dates that can't
be misread (or better, avoid it in real scripts and stick with `format()`).

## Working with a data frame of dates

```r
library(dplyr)

orders <- data.frame(
    order_id = 1:5,
    order_date = c("2024-01-15", "2024-02-20", "2024-02-28", "2024-03-05", "2024-03-15")
)

orders <- orders |>
    mutate(
        order_date = ymd(order_date),
        order_month = floor_date(order_date, unit = "month"),
        days_since_start = as.numeric(order_date - min(order_date))
    )

orders |> count(order_month)
#   order_month n
# 1  2024-01-01 1
# 2  2024-02-01 2
# 3  2024-03-01 2
```

`floor_date(x, unit = "month")` rounds every date down to the first of its
month — the standard way to group daily data into monthly buckets before a
`group_by()`/`count()`. `round_date()` and `ceiling_date()` are the
nearest-value and round-up equivalents.

## Time zones

```r
now_ny <- ymd_hms("2024-06-01 12:00:00", tz = "America/New_York")
with_tz(now_ny, tzone = "Asia/Kolkata")
# [1] "2024-06-01 21:30:00 IST"
```

`with_tz()` converts a moment in time to a different zone's *clock display*
without changing the underlying instant — the correct operation for "what
time is it there right now," as opposed to `force_tz()`, which keeps the
same clock numbers but reinterprets them in a different zone (a much rarer
and more error-prone operation, used only when you know a timestamp was
mislabeled).

## lubridate cheat sheet

| Task | Function |
|------|----------|
| Parse `YYYY-MM-DD` | `ymd(x)` |
| Parse `MM/DD/YYYY` | `mdy(x)` |
| Parse `DD/MM/YYYY` | `dmy(x)` |
| Parse with time | `ymd_hms(x)` |
| Extract component | `year(x)`, `month(x)`, `day(x)`, `wday(x, label=TRUE)` |
| Add calendar time | `x + days(n)`, `x + months(n)`, `x + years(n)` |
| Add exact elapsed time | `x + ddays(n)`, `x + dhours(n)` |
| Round to a unit | `floor_date(x, "month")`, `ceiling_date(x, "week")` |
| Format for display | `format(x, "%B %d, %Y")` (reliable; `stamp("example")(x)` can guess wrong) |
| Convert display time zone | `with_tz(x, "Zone/Name")` |

## Exercise

Given `signups <- c("2024-01-05", "2024-01-20", "2024-02-14", "2024-02-15",
"2024-03-01")`: parse them with `ymd()`, then use `floor_date()` with
`unit = "month"` and `dplyr::count()` to report how many signups happened
per month. Separately, compute how many *calendar days* elapsed between
the first and last signup using period subtraction, and compare that to
using `difftime()` on the same two dates — confirm they agree for this
case, then explain in a comment why periods and durations could disagree
if the range crossed a daylight saving transition.
