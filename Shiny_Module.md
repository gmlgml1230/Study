#Shiny Module

###개요
RDMA의 UI, Server Code는 총 923줄로 이뤄져있으며, 총 4개의 탭이 존재 합니다.
각 탭마다 같은 기능을하고있는 부분이 존재하며 해당 기능의 코드를 각 탭에 추가하는 식으로 작성되어있습니다. 그렇기에 코드를 가독성이 좋지 않으므로 이를 해결하기위해 Shiny Module를 사용하게되었습니다.

###R Module
Shiny Module에 앞서 R의 함수를 Module로 만드는 작업
```
library(modules)

DB <- modules::module({
  # 외부 함수를 사용하기 위해선 Module 함수 안에서 Import를 해주어야함
  import("RMySQL")
  import("DBI")

  host <- "[IP]"

  # 데이터 조회
  db.select <- function(table){
    con <- RMySQL::dbConnect(MySQL(), user = "[user]", password = "[password]", dbname = "[dbname]", host = host, port = [port])
    tryCatch({
      data <- DBI::dbGetQuery(con, paste0("select * from ", table))
      RMySQL::dbDisconnect(con)
      print(paste("DB 조회 완료", print(Sys.time())))
      return(data)
    },
    error = function(e){
      RMySQL::dbDisconnect(con)
      print(e)
    })
  }

  # 테이블 덮어쓰기
  db. <- function(name,data){
    con <- RMySQL::dbConnect(MySQL(), user = "[user]", password = "[password]", dbname = "[dbname]", host = host, port = [port])
    tryCatch({
      DBI::dbWriteTable(con, name, data, overwrite = TRUE)
      RMySQL::dbDisconnect(con)
      print(paste("DB Insert 생성 완료", print(Sys.time())))
    },
    error = function(e){
      RMySQL::dbDisconnect(con)
      print(e)
    })
  }

  # 해당 날짜에 포함되는 데이터를 지운 후 삽입
  db.append <- function(name,data,startdate = NULL,enddate = NULL){
    con <- RMySQL::dbConnect(MySQL(), user = "[user]", password = "[password]", dbname = "[dbname]", host = host, port = [port])
    tryCatch({
      if(is.null(startdate)){dbGetQuery(con, paste0("delete from ", name, " where Day >='", startdate, "' AND Day <='", enddate, "'"))}
      DBI::dbWriteTable(con, name, data, append = TRUE)
      RMySQL::dbDisconnect(con)
      print(paste("DB Insert 완료", print(Sys.time())))
    },
    error = function(e){
      RMySQL::dbDisconnect(con)
      print(e)
    })
  }
})
```

```
DB$db.select
DB$db.overwrite
DB$db.append
```

###R Shiny Module
- Module Function
```
library(modules)

shiny_module <-modules::module({
  import("shiny")
  
  # UI Function
  shiny_module.func <- function(id.chr){
    ns <- NS(id.chr)
    
    tagList(
      actionButton(ns("ok"), "OK"),
      verbatimTextOutput(ns("text"))
    )
  }
  
  
  # Server Function
  shiny_module_server.func <- function(input, output, session, ans){
    observeEvent(input$ok, {
      output$text <- renderText({ans})
    })
  }
})
```

- UI function
```
source("~/ex_shiny/shiny_module.R")

ui <- fluidPage(
  titlePanel("Tabsets"),
  sidebarLayout(
    sidebarPanel(
    ),
    mainPanel(
      tabsetPanel(
        tabPanel("text1", shiny_module$shiny_module.func("text1")),
        tabPanel("text2", shiny_module$shiny_module.func("text2")),
        tabPanel("text3", shiny_module$shiny_module.func("text3"))
      )
    )
  )
)
```

- Server function
```
source("~/ex_shiny/shiny_module.R")

server <- function(input, output) {
  callModule(shiny_module$shiny_module_server.func, "text1", "안녕하세요")
  callModule(shiny_module$shiny_module_server.func, "text2", "모두들")
  callModule(shiny_module$shiny_module_server.func, "text3", "행복하세요")
}
```

###R Shiny Module 심화 예제
- Module Function
```
library(shiny)
library(shinyWidgets)

source("[DB Script path]")

# ==============================================================
# UI
# ==============================================================

report_info_ui.func <- function(id.chr,campaign_list.vec,reportlistorder.vec,reportmetric.vec,reportdimension.vec){
  ns <- NS(id.chr)

  tagList(
    shinyWidgets::materialSwitch(ns("reportinfo"), "Report Info", status = "info"),
    conditionalPanel(condition = paste0("input['", ns("reportinfo"), "'] == true"),
                     wellPanel(
                       selectInput(inputId = ns("campaignlist"), label = "Campaign Name", choices = campaign_list.vec),
                       textInput(inputId = ns("reportname"), label = "Report Name", value = ""),
                       selectInput(inputId = ns("reportdimension"), label = "Select Dimension", choices = reportdimension.vec, multiple = TRUE),
                       selectInput(inputId = ns("reportmetric"), label = "Select Metric", choices = reportmetric.vec, multiple = TRUE),
                       selectInput(inputId = ns("reportorder"), label = "Select OrderBy", choices = reportlistorder.vec),
                       actionButton(inputId = ns( "reportdb"), label = "Submit"),
                       verbatimTextOutput(ns("test"))
                     )
    )
  )
}



# ==============================================================
# Server
# ==============================================================

report_info_server.func <- function(input, output, session, campaign_name.chr, reportname){
  str.func <- function(text.chr){
    return(strsplit(text.chr, ",") %>% unlist())
  }
  report_info.df <- DB$db.select(paste0(campaign_name.chr, "_report_setting kia_report_setting where reportname = '", reportname, "'"))
  updateSelectInput(session, inputId = "campaignlist", selected = report_info.df$campaign)
  updateTextInput(session, inputId = "reportname", value = report_info.df$reportname)
  updateSelectInput(session, inputId = "reportdimension", selected = str.func(report_info.df$dimension))
  updateSelectInput(session, inputId = "reportmetric", selected = str.func(report_info.df$metric))
  updateSelectInput(session, inputId = "reportorder", selected = str.func(report_info.df$orderby))
}

report_info_modi_server.func <- function(input, output, session, func, func2, func3, func4, func5){
  observeEvent(input$reportdb, {
    campaign <- input$campaignlist
    reportname <- input$reportname
    dimension <- func(input$reportdimension)
    metric2 <- func2(input$reportmetric)
    metric <- func3(input$reportmetric)
    query <- func4(dimension, metric2, campaign)
    orderby <- input$reportorder
    settingdata <- data.frame(campaign, reportname, dimension, metric, orderby, query)
    # output$test <- renderText({c(campaign, reportname, dimension, metric, orderby, query)})
    func5("kia_report_setting", settingdata)
    # updateSelectInput(session, inputId = "reportlist", choices = reportlistupdate.func()$reportname)
    showModal(modalDialog(title = "Important message", "Report 생성 완료!"))

  })
}
```
- UI function
```
report_info_ui.func("report_info", campaign_list, reportorderlist, reportmetriclist, reportdimensionlist)
```

- Server function
```
callModule(report_info_server.func, "report_info", "kia", input$reportlist)
```
