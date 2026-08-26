# 10 · Project: Shiny Sales Dashboard

This project pulls together everything from Level 3: `dplyr` filtering
and grouping, `ggplot2` for the chart, and `shiny` for interactivity —
built into one small but complete dashboard with a region filter, a date
range, a summary table, and a bar chart, all driven by reactive
expressions.

## The data

```csv
date,region,product,units,revenue
2024-01-01,East,Widget,10,200
2024-01-01,West,Gadget,5,150
2024-01-02,East,Gadget,8,240
2024-01-02,West,Widget,12,240
2024-01-03,East,Widget,7,140
2024-01-03,West,Gadget,9,270
2024-01-04,East,Gizmo,4,180
2024-01-04,West,Widget,15,300
```

Save this as `data.csv` alongside `app.R`.

## The full app

```r
library(shiny)
library(dplyr)
library(ggplot2)

sales <- read.csv("data.csv", stringsAsFactors = FALSE)
sales$date <- as.Date(sales$date)

ui <- fluidPage(
  titlePanel("Sales Dashboard"),
  sidebarLayout(
    sidebarPanel(
      selectInput("region", "Region", choices = c("All", unique(sales$region))),
      dateRangeInput("dates", "Date range",
        start = min(sales$date), end = max(sales$date))
    ),
    mainPanel(
      textOutput("total_revenue"),
      tableOutput("summary_table"),
      plotOutput("revenue_plot")
    )
  )
)

server <- function(input, output, session) {
  filtered <- reactive({
    df <- sales %>% filter(date >= input$dates[1], date <= input$dates[2])
    if (input$region != "All") df <- df %>% filter(region == input$region)
    df
  })

  output$total_revenue <- renderText({
    paste("Total revenue:", sum(filtered()$revenue))
  })

  output$summary_table <- renderTable({
    filtered() %>% group_by(product) %>%
      summarise(units = sum(units), revenue = sum(revenue), .groups = "drop")
  })

  output$revenue_plot <- renderPlot({
    ggplot(filtered(), aes(x = date, y = revenue, fill = product)) +
      geom_col() + theme_minimal()
  })
}

shinyApp(ui, server)
```

## Design notes

`filtered()` is the single `reactive()` all three outputs depend on — the
region and date filtering logic is written exactly once, and every output
downstream automatically sees the same filtered slice. This is the same
"compute once, reuse everywhere" pattern from Module 5: writing the
`filter()` calls three separate times, once per output, would triple the
maintenance burden and risk the three outputs drifting out of sync if one
copy got edited and the others didn't.

The `if (input$region != "All")` branch inside `filtered()` is a common
dashboard pattern — an "All" option that means "skip this filter
entirely" rather than being a real value to filter on. It's tempting to
instead add `"All"` as a literal value in the `region` column, but that
would corrupt the underlying data just to make one UI control simpler.

## Running it

Testing a Shiny app's logic doesn't require a browser — `testServer()`
runs the server function directly and lets you simulate widget input:

```r
source("app.R", local = (env <- new.env()))
testServer(env$server, {
  session$setInputs(region = "All", dates = c(as.Date("2024-01-01"), as.Date("2024-01-04")))
  output$total_revenue
  # [1] "Total revenue: 1720"

  session$setInputs(region = "East")
  output$total_revenue
  # [1] "Total revenue: 760"
})
```

Both numbers check out against the raw data: summing all 8 rows' revenue
gives 1720; summing only the `East` rows (200 + 240 + 140 + 180) gives
760. Changing `input$region` from `"All"` to `"East"` automatically
recomputed `filtered()` and every output built on top of it — no manual
recalculation code, which is the entire point of Shiny's reactive model.

To actually see the dashboard, run `shiny::runApp("path/to/app_dir")`
from an interactive R session, which launches the app in a browser tab
with live sliders and dropdowns.

## Stretch goals

- Add a `selectInput` for `product` alongside the existing `region`
  filter, wire it into the same `filtered()` reactive, and confirm the
  table and plot both respect it without any other code changes.
- Replace the static bar chart with a `plotOutput` that switches between
  a bar chart and a line chart based on a `radioButtons()` input —
  practice branching *inside* a `render*()` block based on another input.
- Add a CSV download button (`downloadHandler()` + `downloadButton()`)
  that lets the user export the currently filtered data, not the full
  dataset.
- Write a `testServer()` test suite covering at least three input
  combinations (all regions, one specific region, a narrowed date range)
  and assert on `output$total_revenue` for each, so a future edit to the
  filtering logic can be checked against these expected totals.

Completing this project means you're ready for **Level 4 · Expert**.
