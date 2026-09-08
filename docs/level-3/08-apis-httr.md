# 08 · Calling APIs with httr

Most real-world data doesn't arrive as a tidy CSV — it comes from a web
API you query over HTTP. This module uses `httr` to make GET and POST
requests against a live test API
(`https://jsonplaceholder.typicode.com`), parse JSON responses, and
handle the errors that real APIs eventually throw at you.

```r
library(httr)
library(jsonlite)
```

## A basic GET request

```r
resp <- GET("https://jsonplaceholder.typicode.com/users/1")
status_code(resp)
# [1] 200

data <- content(resp, as = "parsed", type = "application/json")
data$name
# [1] "Leanne Graham"
data$email
# [1] "Sincere@april.biz"
```

`GET()` sends the request and returns a response object — `status_code()`
reads the HTTP status without touching the body, and `content()` parses
the body into an R structure. `as = "parsed"` with `type =
"application/json"` gives you nested lists that mirror the JSON's
structure directly; `as = "text"` (not shown) gives you the raw JSON
string if you'd rather parse it yourself with `jsonlite::fromJSON()`.

## Passing query parameters

```r
resp2 <- GET("https://jsonplaceholder.typicode.com/posts", query = list(userId = 1))
posts <- content(resp2, as = "parsed", type = "application/json")
length(posts)
# [1] 10
```

`query = list(...)` builds the URL's `?key=value&...` query string for
you, correctly encoding special characters — safer and more readable than
building the string by hand with `paste0()`.

## Sending data with POST

```r
resp3 <- POST(
  "https://jsonplaceholder.typicode.com/posts",
  body = list(title = "hello", body = "world", userId = 1),
  encode = "json"
)
status_code(resp3)
# [1] 201

content(resp3, as = "parsed")
# $title
# [1] "hello"
# $body
# [1] "world"
# $userId
# [1] 1
# $id
# [1] 101
```

`encode = "json"` tells `httr` to serialize the `body` list as a JSON
request body and set the right `Content-Type` header automatically — the
single most common mistake with `POST()` is sending a plain list without
specifying an encoding and having the API silently reject or
misinterpret the payload. A `201` status confirms the resource was
created; this particular test API echoes back the payload plus a
generated `id`, which is typical of REST APIs that create resources.

## Handling errors: don't assume 200

```r
resp4 <- GET("https://jsonplaceholder.typicode.com/nonexistent/999")
status_code(resp4)
# [1] 404
```

**Trap:** `GET()` does *not* raise an R error for a 404, 500, or any
other non-2xx HTTP status — it returns a normal response object exactly
like a successful call, and `content()` on it will happily parse
whatever the server sent back for the error page. Code that calls
`content()` immediately after `GET()` without checking `status_code()`
first can silently process an error page as if it were real data.

```r
stop_for_status(resp4)
# Error: Not Found (HTTP 404)
```

`stop_for_status()` converts a non-2xx response into an actual R error —
call it right after every request whose success you can't otherwise
verify, so a failed API call stops your script loudly instead of feeding
garbage into whatever comes next. `warn_for_status()` (not shown) is the
softer variant: a warning instead of a hard stop, for calls you want to
continue past.

## Setting headers (authentication, content type)

```r
GET(
  "https://api.example.com/data",
  add_headers(Authorization = paste("Bearer", Sys.getenv("API_TOKEN")))
)
```

**Trap:** never hard-code an API key or token directly in a script.
`Sys.getenv("API_TOKEN")` reads it from an environment variable (set once
in your shell profile or an untracked `.Renviron` file), keeping secrets
out of version control — a script that runs correctly with a hard-coded
key is one accidental `git commit` away from leaking it.

## Cheat sheet

| Task | Function |
|---|---|
| GET request | `GET(url)` |
| POST request with JSON body | `POST(url, body = list(...), encode = "json")` |
| Add query string parameters | `GET(url, query = list(...))` |
| Add custom headers (auth, content-type) | `add_headers(...)` |
| Read HTTP status code | `status_code(resp)` |
| Parse JSON response body | `content(resp, as = "parsed", type = "application/json")` |
| Raise an R error on non-2xx status | `stop_for_status(resp)` |
| Warn (don't stop) on non-2xx status | `warn_for_status(resp)` |
| Keep a secret out of source code | `Sys.getenv("VAR_NAME")` |

## How It Actually Works

`httr`/`httr2` build an HTTP request by constructing a request object
(method, URL, headers, body) and handing it to **libcurl** — the same C
library `curl` the command-line tool and countless other languages use —
which R links against via a compiled interface (the `curl` package).
libcurl handles the actual TCP connection, TLS handshake, and HTTP
protocol framing; R's role is just building the request object correctly
and parsing the raw response bytes libcurl hands back.

`GET()`/`POST()` are synchronous and **blocking**: the R process pauses at
that line, doing nothing else, until the full response arrives or the
request times out — there's no event loop unless you explicitly use an
async-capable package. Response parsing (`content(resp, "parsed")`)
inspects the `Content-Type` header to decide *how* to interpret the raw
response bytes — JSON gets fed through `jsonlite`'s recursive descent
parser (which builds R lists/vectors by walking the JSON token stream),
while a JSON array of uniform objects gets coerced into a data frame by
jsonlite auto-detecting that every object shares the same keys. Rate-limit
handling you write yourself (checking `status_code() == 429` and
`Sys.sleep()`) exists because none of this HTTP machinery has any built-in
notion of API-specific rate limits — that logic is exactly as manual as
retry logic in any other language's HTTP client.

## Exercise

1. GET a resource from `jsonplaceholder.typicode.com` (any of `/users`,
   `/posts`, `/comments`), parse the JSON, and extract one field from
   each item into a plain vector or data frame.
2. Request a URL you know will 404 and confirm `status_code()` returns
   404 without `GET()` itself raising an error — then wrap the same call
   with `stop_for_status()` and confirm it does raise one.
3. Write a small function `safe_get(url)` that performs a `GET()`,
   calls `stop_for_status()`, and returns the parsed JSON — with a
   `tryCatch()` around the whole thing that returns `NULL` and prints a
   clear message on failure instead of crashing the caller.
