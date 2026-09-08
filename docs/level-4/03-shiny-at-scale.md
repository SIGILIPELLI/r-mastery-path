# 03 · Shiny at Scale (Modules)

A Shiny app that started as one `app.R` file gets unmanageable fast once
it has a dozen inputs feeding a dozen outputs — every `input$x` and
`output$y` lives in one shared global namespace, so two sliders named
`input$range` anywhere in the app collide. Shiny modules fix this by
wrapping a chunk of UI and server logic in its own namespace, so it can
be written, tested, and reused independently of the rest of the app.

```r
suppressMessages(library(shiny))
```

## The problem modules solve

```r
# Without modules — two "counter" features can't coexist:
ui <- fluidPage(
  actionButton("increment", "Add"),
  textOutput("count")
)
server <- function(input, output, session) {
  count <- reactiveVal(0)
  observeEvent(input$increment, count(count() + 1))
  output$count <- renderText(count())
}
```

Add a second counter to the same page and `input$increment` /
`output$count` collide — the second definition silently overwrites the
first. A module namespaces both sides so any number of counters can
coexist.

## Writing a module: UI and server halves

A module is two functions with a shared prefix — `counterUI()` and
`counterServer()` — where the UI half wraps every input/output ID in
`NS(id)` and the server half is entered with `moduleServer()`:

```r
counterUI <- function(id) {
  ns <- NS(id)
  tagList(
    actionButton(ns("increment"), "Add"),
    textOutput(ns("count"))
  )
}

counterServer <- function(id, start = 0) {
  moduleServer(id, function(input, output, session) {
    count <- reactiveVal(start)
    observeEvent(input$increment, count(count() + 1))
    output$count <- renderText(count())
    count  # expose the reactive so a parent can read it
  })
}
```

`ns("increment")` turns `"increment"` into something like
`"counterA-increment"` — the module's `id` becomes a prefix baked into
the actual DOM element ID, so two calls to `counterUI("counterA")` and
`counterUI("counterB")` never collide even though the module code inside
only ever refers to the short name `"increment"`.

## Composing modules in an app

```r
ui <- fluidPage(
  fluidRow(
    column(6, h4("Counter A"), counterUI("counterA")),
    column(6, h4("Counter B"), counterUI("counterB"))
  ),
  textOutput("total")
)

server <- function(input, output, session) {
  a <- counterServer("counterA", start = 0)
  b <- counterServer("counterB", start = 10)
  output$total <- renderText(paste("Total:", a() + b()))
}
```

The parent never touches `input$counterA-increment` directly — it calls
`counterServer("counterA")` once, gets back the reactive value the
module chose to expose, and combines it with the other module's value.
The module's internals stay private; only what it explicitly returns is
visible outside.

## Testing module logic without a browser

`shiny::testServer()` runs a module's server function in isolation,
letting you simulate inputs and assert on internal reactive state — no
browser, no running app, so this is normal `testthat` code:

```r
testServer(counterServer, args = list(start = 5), {
  expect_equal(count(), 5)
  session$setInputs(increment = 1)
  expect_equal(count(), 6)
  session$setInputs(increment = 2)  # value doesn't matter, only that it fired
  expect_equal(count(), 7)
})
```

Running this file with `testthat::test_file()`:

```
Test passed
```

`session$setInputs()` simulates a user clicking/typing and flushes
Shiny's reactive graph exactly as a real browser event would — `count`
and any other object defined inside `moduleServer()`'s function body are
directly visible inside the `testServer()` block, which is the main
reason to pull logic into a module in the first place: it becomes
testable outside of `shinytest2`'s slower, browser-driving tests.

## R-specific traps

**Forgetting `ns()` on an input/output ID inside the UI function** is the
single most common module bug — the ID leaks out unnamespaced, so it
either collides with a same-named ID elsewhere or simply never matches
what the server half is listening for (which is *always* the short,
un-namespaced name — `moduleServer()` handles the translation for you
automatically).

**Reactives returned from a module must be called as functions.** `a <-
counterServer("counterA")` gives you a reactive expression, not a value
— using `a` directly in an expression like `a + b()` is a bug; you need
`a() + b()`. This is the same reactive-vs-value distinction from Level 3,
just easier to trip over when the reactive crossed a module boundary.

**Module IDs must be unique strings, not reused across renders.** If a
module is created inside a `renderUI()` or a loop with a computed ID,
make sure the ID is stable across re-renders (e.g. derived from a data
key, not a row index that can shift) — otherwise Shiny treats what should
be the same module instance as a brand new one on every redraw, silently
losing its state.

## Cheat sheet

| Task | Function |
|---|---|
| Define a module's UI half | `function(id) { ns <- NS(id); tagList(...) }` |
| Namespace an ID inside the UI half | `ns("some_id")` |
| Define a module's server half | `moduleServer(id, function(input, output, session) {...})` |
| Instantiate a module in an app | `counterUI("counterA")` / `counterServer("counterA")` |
| Expose a value to the parent | `return()` (or last expression) a reactive from the server function |
| Test a module without a browser | `testServer(counterServer, args = list(...), { session$setInputs(...); expect_equal(...) })` |
| Simulate a user input in a test | `session$setInputs(name = value)` |

## How It Actually Works

Shiny modules (`moduleServer()`) solve a real namespace-collision problem
mechanically, not just organizationally: `NS(id)` generates a prefixing
function that turns `"plot"` into `"myModule-plot"` in the HTML `id`
attribute, and `moduleServer()` wraps your module's server logic in its
*own* nested environment, so `input$plot` inside the module resolves
against that module's own scoped reactive values rather than the app's
global `input` — two instances of the same module can coexist because each
call to `moduleServer()` creates an independent closure over a freshly
generated namespace, not because Shiny does anything special at the HTTP
layer.

Scaling a Shiny app under real concurrent load runs into the same
single-threaded-R-process ceiling as plumber: one R process serves one
user's reactive graph updates at a time by default, so production
deployments (Shiny Server Pro, Posit Connect, or a container orchestrator)
run **multiple R processes**, each holding independent copies of the app,
behind a load balancer that pins each browser session (via sticky
sessions) to the one R process holding that session's actual live reactive
state — a session can't be transparently moved between processes mid-flight
because its reactive graph and any accumulated in-memory data live only in
that one process's memory.

## Exercise

1. Write a `filterUI(id)` / `filterServer(id, data)` module pair that
   shows a `selectInput()` of column names from `data` and returns a
   reactive vector of that column's values.
2. Instantiate two independent copies of your `filterUI`/`filterServer`
   module for two different data frames in the same app, and confirm in
   the UI that selecting a column in one doesn't affect the other.
3. Write a `testServer()` test for your `filterServer` module that sets
   the selected column via `session$setInputs()` and asserts the
   returned reactive matches the expected column values, with no browser
   involved.
