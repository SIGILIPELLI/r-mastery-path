# 05 · Shiny Basics

Every R script so far runs top to bottom and stops. **Shiny** turns an R
script into an interactive web app: a user moves a slider or picks a
dropdown in a browser, and the parts of the app that depend on that input
recompute automatically — no manual "run again" required. This module
covers the core mental model (UI + server + reactivity) and builds a
small working app.

```r
library(shiny)
```

## The two halves of every Shiny app

```r
ui <- fluidPage(
  sliderInput("n", "Number of points", 10, 100, 50),
  textOutput("summary_text"),
  plotOutput("scatter")
)

server <- function(input, output, session) {
  data <- reactive({
    set.seed(1)
    data.frame(x = rnorm(input$n), y = rnorm(input$n))
  })

  output$summary_text <- renderText({
    paste("Showing", nrow(data()), "points")
  })

  output$scatter <- renderPlot({
    plot(data()$x, data()$y)
  })
}

shinyApp(ui, server)
```

`ui` is a nested tree of layout and input/output *placeholders* — it
describes what widgets exist and where output should appear, but holds no
logic. `server` is a function that wires those placeholders to actual
computation: `input$n` reads the current slider value, `output$scatter <-
renderPlot({...})` says "whenever something this block depends on
changes, redraw this plot." `shinyApp(ui, server)` starts the app,
launching a browser tab if run interactively.

## Reactivity: the core idea

`input$n`, `reactive({...})` expressions, and `output$*` assignments form
a dependency graph, not a sequence of steps. `data()` in the example above
is a `reactive()` expression — think of it as a value that recomputes
itself automatically whenever an input it reads (`input$n`) changes,
*and* automatically re-notifies anything that reads `data()` in turn. You
never write "when the slider changes, update the plot" explicitly — Shiny
infers that wiring from which reactive values each block reads.

**Trap:** a `reactive()` expression is not a regular function — calling
it outside of a reactive context (a `render*()` block, another
`reactive()`, or an `observe()`) raises an error, because Shiny needs to
be inside that graph to know what to re-run when things change. Don't try
to call `data()` at the top level of `server()`, only inside blocks that
are themselves reactive.

## Testing server logic without a browser

Manually clicking through an app to check its logic doesn't scale, and
isn't reproducible. `shiny::testServer()` runs the `server` function in
isolation, letting you simulate input changes and assert on the resulting
reactive values directly — the same style of test you'd write for any
other function, applied to reactive logic:

```r
testServer(server, {
  session$setInputs(n = 20)
  nrow(data())
  # [1] 20
  output$summary_text
  # [1] "Showing 20 points"

  session$setInputs(n = 40)
  output$summary_text
  # [1] "Showing 40 points"
})
```

`session$setInputs()` simulates the user changing a widget; every
`reactive()` and `output$*` that depends on that input recomputes exactly
as it would in a live app, without a browser or a running server. This is
the practical way to verify reactive logic is correct before ever opening
a browser tab.

## Common UI building blocks

```r
fluidPage(
  titlePanel("My App"),
  sidebarLayout(
    sidebarPanel(
      selectInput("category", "Category", choices = c("A", "B", "C")),
      numericInput("threshold", "Threshold", value = 10)
    ),
    mainPanel(
      tableOutput("results"),
      plotOutput("chart")
    )
  )
)
```

`sidebarLayout()` is the most common starting layout — controls on the
left, results on the right. Each `*Input()` function (`sliderInput()`,
`selectInput()`, `numericInput()`, `textInput()`, `checkboxInput()`)
creates a widget whose current value shows up as `input$<id>` in the
server; each `*Output()` function (`textOutput()`, `plotOutput()`,
`tableOutput()`) is a placeholder that a matching `render*()` call in the
server fills in.

## Avoiding redundant computation

```r
server <- function(input, output, session) {
  filtered <- reactive({
    subset(mtcars, mpg > input$threshold)
  })

  output$results <- renderTable(filtered())
  output$chart <- renderPlot(hist(filtered()$mpg))
}
```

Both `output$results` and `output$chart` depend on `filtered()`, but
`filtered()` is computed **once** per input change and its cached result
is reused by both outputs — this is why filtering or expensive
computation belongs in a shared `reactive()` rather than being repeated
inside every `render*()` block that needs it. Duplicating the `subset()`
call inside both render blocks would recompute it twice on every slider
move for no benefit.

## Cheat sheet

| Task | Function |
|---|---|
| Define the app's layout | `fluidPage(...)` |
| Read a widget's current value | `input$<id>` |
| Shared, cached, auto-updating value | `reactive({...})` |
| Render text output | `renderText({...})` / `textOutput("id")` |
| Render a plot | `renderPlot({...})` / `plotOutput("id")` |
| Render a table | `renderTable({...})` / `tableOutput("id")` |
| Start the app | `shinyApp(ui, server)` |
| Test server logic headlessly | `testServer(server, {...})` |
| Simulate a widget change in a test | `session$setInputs(id = value)` |

## Exercise

1. Build a small app with a `selectInput()` for a dataset column and a
   `renderPlot()` histogram of that column, using `mtcars` or a data
   frame of your own.
2. Add a second output that depends on the same filtered/reactive data as
   the first, and confirm (by reasoning about the dependency graph, or by
   adding a `cat()` inside the shared `reactive()`) that the shared
   computation only runs once per input change, not once per output.
3. Write a `testServer()` block for your app that sets at least two
   different input values and asserts the resulting output text or data
   changes accordingly — no browser required.
