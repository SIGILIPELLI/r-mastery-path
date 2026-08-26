# 01 · Advanced Statistical Methods

Level 3 covered `lm()` and `glm()` for modeling a single relationship.
Real analysis work often needs to compare more than two groups at once,
reduce a wide table of correlated columns down to a few meaningful axes,
find structure with no labels at all, or fit a trend to data collected
over time. This module covers `aov()` (ANOVA), `prcomp()` (PCA),
`kmeans()` (clustering), and a first look at time series.

```r
set.seed(42)
suppressMessages(library(broom))
```

## Comparing more than two groups: ANOVA

A `t.test()` compares exactly two group means. With three or more groups,
running pairwise t-tests inflates your false-positive rate — `aov()`
tests all groups at once with a single p-value:

```r
groups <- factor(rep(c("A", "B", "C"), each = 20))
score <- c(rnorm(20, 50, 5), rnorm(20, 55, 5), rnorm(20, 60, 5))
df <- data.frame(groups, score)

fit <- aov(score ~ groups, data = df)
summary(fit)
#             Df Sum Sq Mean Sq F value   Pr(>F)
# groups       2  861.2   430.6   12.91 2.37e-05 ***
# Residuals   57 1900.9    33.3
```

`Pr(>F)` of 2.37e-05 says the three group means are very unlikely to be
equal by chance. That's the whole answer ANOVA gives you, though — it
tells you *some* pair differs, not *which* pair.

```r
TukeyHSD(fit)
#         diff       lwr       upr     p adj
# B-A 2.685441 -1.709077  7.079959 0.3125723
# C-A 9.035846  4.641329 13.430364 0.0000206
# C-B 6.350405  1.955888 10.744923 0.0027668
```

`TukeyHSD()` runs every pairwise comparison with the confidence intervals
already corrected for multiple testing — here `C` differs significantly
from both `A` and `B`, but `B` doesn't clearly differ from `A` (its
interval spans zero).

## Reducing dimensions with PCA

Principal Component Analysis re-expresses correlated columns as a smaller
set of uncorrelated "components" that capture most of the original
variance:

```r
set.seed(1)
x1 <- rnorm(100)
x2 <- x1 * 0.8 + rnorm(100, 0, 0.3)  # correlated with x1
x3 <- rnorm(100)                      # independent noise
mat <- data.frame(x1, x2, x3)

pca <- prcomp(mat, scale. = TRUE)
summary(pca)
#                           PC1    PC2     PC3
# Standard deviation     1.3887 1.0000 0.26709
# Proportion of Variance 0.6429 0.3333 0.02378
# Cumulative Proportion  0.6429 0.9762 1.00000
```

PC1 alone captures 64% of the total variance — unsurprising, since `x1`
and `x2` are strongly correlated and PCA folds correlated columns into
fewer axes. Look at the rotation (loadings) to see *why*:

```r
pca$rotation
#          PC1          PC2         PC3
# x1 0.70711606 -0.001427325  0.70709607
# x2 0.70697822  0.019794755 -0.70695825
# x3 0.01298773 -0.999803046 -0.01500628
```

PC1 weights `x1` and `x2` almost equally (0.707 each ≈ 1/√2) and nearly
ignores `x3` — it's essentially "the shared signal between x1 and x2."
PC2 is almost entirely `x3`. **Always set `scale. = TRUE`** unless your
columns are already on the same unit — otherwise a column measured in
larger numbers dominates the components purely because of its scale, not
because it's more informative.

## Finding structure with k-means

Where PCA reduces columns, `kmeans()` groups *rows* into clusters based
on distance, with no labels supplied:

```r
set.seed(2)
pts <- rbind(matrix(rnorm(50, 0, 1), ncol = 2),
             matrix(rnorm(50, 5, 1), ncol = 2))

km <- kmeans(pts, centers = 2)
km$centers
#       [,1]       [,2]
# 1 0.3339737 -0.1956978
# 2 4.9238090  4.8151226

table(km$cluster)
#  1  2
# 25 25
```

`kmeans()` recovered the two generating clusters (centered near `(0,0)`
and `(5,5)`) with a clean 25/25 split, matching how the data was built.
In practice you don't know the true number of clusters ahead of time —
`centers` is a required guess, and choosing it wrong (an "elbow plot" of
within-cluster sum of squares across several values of `centers` is the
usual way to pick) is the most common mistake with this function.

## A first look at time series

```r
set.seed(3)
ts_data <- ts(cumsum(rnorm(50)) + 1:50 * 0.5, frequency = 1)
mod <- lm(as.numeric(ts_data) ~ seq_along(ts_data))
coef(mod)
#        (Intercept) seq_along(ts_data)
#         -2.9053801          0.4866118
```

`ts()` wraps a numeric vector with time metadata (`frequency` records how
many observations make up one cycle — 1 here means no seasonality).
Fitting `lm()` against the index recovers a slope close to the `0.5` drift
built into the simulation; this linear-trend model ignores autocorrelation
between consecutive points, which is why dedicated packages like
`forecast` or `fable` exist for anything beyond a rough first look.

## R-specific traps

**ANOVA assumes equal variance across groups**, just like `t.test(var.equal
= TRUE)` — `aov()` doesn't check this for you. Use `oneway.test()` for a
Welch-corrected version, or `bartlett.test()` to test the assumption
first.

**PCA is sensitive to outliers and scale.** A single extreme row can pull
an entire component toward it, and forgetting `scale. = TRUE` silently
lets whichever column has the largest raw numbers dominate every
component regardless of its actual signal.

**`kmeans()` results depend on random initialization.** Two runs with
different seeds can converge to different cluster assignments (different
local optima), especially with `centers` set too high or too low — always
set a seed for reproducibility, and consider `nstart = 25` to run the
algorithm from multiple random starts and keep the best result.

## Cheat sheet

| Task | Function |
|---|---|
| Compare 3+ group means at once | `aov(y ~ group, data = df)` |
| Post-hoc pairwise comparisons | `TukeyHSD(fit)` |
| Reduce correlated columns to components | `prcomp(df, scale. = TRUE)` |
| See variance explained per component | `summary(pca)` |
| See how columns load onto components | `pca$rotation` |
| Cluster rows with no labels | `kmeans(df, centers = k, nstart = 25)` |
| Wrap a vector with time metadata | `ts(x, frequency = n)` |
| Test equal variance before ANOVA | `bartlett.test(y ~ group, data = df)` |

## Exercise

1. Simulate four groups where three share a mean and one is clearly
   different, run `aov()`, then use `TukeyHSD()` to confirm which
   specific pairs differ and which don't.
2. Build a data frame with five numeric columns where two are strongly
   correlated and three are independent noise, run `prcomp(scale. =
   TRUE)`, and check that the first component's loadings match the
   correlated pair.
3. Run `kmeans()` twice on the same data with different seeds but
   `nstart = 1`, and compare `$centers` — then rerun both with `nstart =
   25` and confirm the results now agree.
