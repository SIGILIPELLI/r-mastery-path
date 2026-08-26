# 02 · Production R with plumber

Everything so far has run in a script or console. `plumber` turns an R
file into an HTTP API by reading special `#*` comments above each
function — so any model or data pipeline you've built can be called over
the network from another service, a dashboard, or a script in any
language.

```r
library(plumber)
```

## Defining routes with roxygen-style comments

`plumber` reads annotations directly above each function to decide the
HTTP method and path. Save this as `api/plumber.R`:

```r
#* Echo a message back
#* @param msg The message to echo
#* @get /echo
function(msg = "") {
  list(msg = msg)
}

#* Sum a vector of numbers
#* @post /sum
function(req) {
  body <- jsonlite::fromJSON(req$postBody)
  list(total = sum(unlist(body$nums)))
}

#* Health check
#* @get /health
function() {
  list(status = "ok", time = as.character(Sys.time()))
}
```

`@get` and `@post` map directly to HTTP verbs. A `@get` route reads its
inputs from query parameters (matched to the function's argument names);
a `@post` route reads the raw JSON body off the special `req` object and
you parse it yourself with `jsonlite::fromJSON()`.

## Running it

```r
pr <- plumb("api/plumber.R")
pr$run(port = 8199, host = "127.0.0.1")
```

From another terminal:

```bash
curl "http://127.0.0.1:8199/echo?msg=hello"
# {"msg":["hello"]}

curl "http://127.0.0.1:8199/health"
# {"status":["ok"],"time":["2026-08-26 11:06:19.8659"]}

curl -X POST -H "Content-Type: application/json" \
  -d '{"nums":[1,2,3,4]}' "http://127.0.0.1:8199/sum"
# {"total":[10]}
```

Every value plumber returns is auto-serialized to JSON, and every scalar
comes back wrapped in an array (`"msg":["hello"]` not `"msg":"hello"`) —
that's `jsonlite`'s default array-safe encoding, not a plumber quirk, and
it's worth knowing before a client written against "always an array"
assumptions breaks the day someone changes `unbox = TRUE`.

## Serving a model as an endpoint

The whole point of a production API is usually to wrap something you've
already built — here, a fitted regression from Level 3:

```r
#* Predict y from x using a pre-fitted model
#* @param x:numeric The input value
#* @get /predict
function(x) {
  x <- as.numeric(x)
  pred <- predict(fit, newdata = data.frame(x = x), interval = "confidence")
  list(fit = pred[1, "fit"], lwr = pred[1, "lwr"], upr = pred[1, "upr"])
}
```

`fit` here is a model object loaded once when the file is sourced (e.g.
via `readRDS("model.rds")` above the route definitions) — plumber loads
the whole file a single time at startup, so any objects you create outside
a route function are computed once and reused across every request,
not recomputed per call.

## Error handling

By default, an uncaught error inside a route returns a generic 500 with
no detail — useful for hiding internals from callers, useless for
debugging:

```r
#* @get /risky
function(x) {
  if (missing(x)) stop("x is required")
  as.numeric(x) * 2
}
```

```bash
curl "http://127.0.0.1:8199/risky"
# {"error":"500 - Internal server error"}
```

Wrap the body in `tryCatch()` to return a structured error instead of the
generic message, and set the response status explicitly with `res`:

```r
#* @get /risky
function(x, res) {
  tryCatch({
    if (missing(x)) stop("x is required")
    list(result = as.numeric(x) * 2)
  }, error = function(e) {
    res$status <- 400
    list(error = conditionMessage(e))
  })
}
```

## R-specific traps

**Global objects loaded at file-source time are shared across concurrent
requests.** If a route mutates a global variable (rather than only
reading it), two simultaneous requests can race — plumber by default runs
single-threaded per process, so within one process this is usually safe,
but don't assume it once you scale to multiple worker processes behind a
load balancer.

**`@param` type annotations don't actually validate or coerce for you** in
older plumber versions the way they look like they should — `x` above
arrives as a **character string** from the query string even with `x:numeric`
in some setups, which is why the handler still calls `as.numeric(x)`
explicitly. Don't skip the explicit conversion just because the
annotation looks typed.

**Restarting the R session drops all in-memory state**, including any
model loaded via `readRDS()` at the top of the file — a production
deployment needs that load to happen automatically on every process
start (which plumber does, since it re-sources the file when a new
process boots), not something that only works because your local session
happened to still have it in memory.

## Cheat sheet

| Task | Annotation / call |
|---|---|
| Define a GET route | `#* @get /path` |
| Define a POST route | `#* @post /path` |
| Access raw request body | function arg `req`, then `jsonlite::fromJSON(req$postBody)` |
| Access response object to set status | function arg `res` |
| Load a plumber file without running it | `plumb("file.R")` |
| Start the server | `pr$run(port = 8199, host = "127.0.0.1")` |
| Return a non-200 status | `res$status <- 400` inside the handler |
| Catch errors inside a route | wrap handler body in `tryCatch()` |

## Exercise

1. Write a `plumber.R` with a `@get /square` route that takes a numeric
   query parameter and returns its square, then hit it with `curl` and
   confirm the JSON response.
2. Add a `@post /classify` route that reads a JSON body with a `values`
   array and returns whether each value is above or below the mean of
   the array — parse the body with `jsonlite::fromJSON()`.
3. Take the `/risky` example above, deliberately trigger the error path
   with a missing parameter, and confirm the response comes back with
   HTTP status 400 and a JSON `error` field instead of a generic 500.
