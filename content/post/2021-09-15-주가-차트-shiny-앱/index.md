---
title: 주가 차트 Shiny 앱
author: 단호진
date: '2021-09-15'
slug: 주가-차트-shiny-앱
categories: []
tags:
  - shiny app
---

# 주가 차트 Shiny 앱

종목을 선택하면 주가 일봉 차트가 도시되는 앱을 개발하였다. rstudio의 [Use reactive expressions](https://shiny.rstudio.com/tutorial/written-tutorial/lesson6/) 튜토리얼을 참고하였으며, 앱의 서비스 모습은 다음과 같다. 

![stockviz app](/images/stockviz.png)

shiny 앱의 기본 구조는 ui와 server 함수를 작성하여 shinyApp 함수에 인수로 제공하면 된다. ui 함수에는 사용자와 상호 작용을 위한 Input으로 끝나는 위젯이나 보여주고자 하는 사항을 담은 Output 위젯이 있다. server 함수 내 render로 시작하는 함수는 Input과 Output 변수에 접근하여 반응형 앱이 되도록 한다. 

앱은 rstudio 환경에서 실행하거나 프로젝트 루트에서 R -e "shiny::runApp()" 명령어를 이용하여 실행할 수 있다. 짧은 코드이므로 자세한 설명 없이 코드 전체를 남겨 두겠다. 


```r
library(shiny)
library(tidyverse)
library(tidyquant)

data("FANG")

fang <- FANG
symbols <- fang %>% distinct(symbol)

ui <- fluidPage(
  titlePanel("stockVis"),
  sidebarLayout(
    sidebarPanel(
      helpText(paste("Select a stock to examine.",
                     "Information will be collected from Yahoo finance.")),
      selectInput("symbol",
                  "Symbol",
                  choices = symbols),
      dateRangeInput("date_range",
                     "Date range",
                     start = "2015-03-01",
                     end = "2015-06-01",
                     min = "2013-01-01",
                     max = "2017-01-01"),
      br(),
      br(),
      # checkboxInput("scale_y_log",
      #               "Plot y axis on log scale"),
      checkboxInput("bbands",
                    "Bollinger bands"),
    ),
    mainPanel(
      plotOutput(outputId = "closing_price")
    )
  )
)

server <- function(input, output) {
  output$closing_price <- renderPlot({
    p <- fang %>%
      filter(symbol == input$symbol,
             date %within% interval(start = as_date(input$date_range[[1]]),
                                    end = as_date(input$date_range[[2]]))) %>%
      ggplot(aes(x = date, y = close,
                 open = open, high = high, low = low, close = close)) +
      # geom_line() +
      # geom_barchart() +
      geom_candlestick(colour_up = "#E02D23", colour_down = "#377CE5",
                       fill_up = "#E02D23", fill_down = "#377CE5") +
      theme_tq()
    if(input$bbands) {
      p <- p + geom_bbands(ma_fun = SMA, sd = 2, n = 20, linetype = 5)
    }
    # if(input$scale_y_log) {
    #   p <- p + scale_y_log10()
    # }
    p
  })
}

shinyApp(ui, server)
```
