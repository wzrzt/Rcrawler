杭州至北京2018年1月1日余票
================
Rain.Wei
2017-12-25 11:18:02

启动Selenium Web Driver
-----------------------

``` r
# Run a server for example using Docker
# docker run -d -p 4445:4444 selenium/standalone-firefox:2.53.1
# Use a debug image with a VNC viewer if you wish to view the browser
# docker run -d -p 5901:5900 -p 127.0.0.1:4445:4444 --link http-server selenium/standalone-firefox-debug:2.53.1
# See Docker vignette for more detail or run a Selenium Server manually.
remDr <- remoteDriver(remoteServerAddr = "localhost" 
                      , port = 4445L 
                      , browserName = "chrome" 
                      ) 
remDr$open(silent = TRUE)  
```

``` r
remDr$navigate('https://kyfw.12306.cn/otn/leftTicket/init') 
```

Input Start Station and Arrival Station
=======================================

``` r
Sys.sleep(1 + runif(1)) 
elem.fromStation <- remDr$findElement(using = 'xpath', '//*[@id="fromStationText"]')
elem.fromStation$clickElement() 
elem.fromStation$sendKeysToElement(list("杭州"))
Sys.sleep(1 + runif(1))
elem.city.hangzhou <- remDr$findElement(using = 'xpath', '//*[@id="citem_1"]/span[1]')
elem.city.hangzhou$clickElement() 
Sys.sleep(1 + runif(1))
elem.toStation <- remDr$findElement(using = 'xpath', '//*[@id="toStationText"]') 
elem.toStation$sendKeysToElement(list("北京")) 
Sys.sleep(1 + runif(1)) 
elem.city.beijing <- remDr$findElement(using = 'xpath', '//*[@id="citem_2"]') 
elem.city.beijing$clickElement() 
```

Input Start Date
================

``` r
Sys.sleep(1 + runif(1)) 
elem.date_input <- remDr$findElement(using = 'xpath', '//*[@id="train_date"]') 
elem.date_input$clickElement() 
Sys.sleep(1 + runif(1)) 
elem.date_20180101 <- remDr$findElement(using = 'xpath', 
                                        '/html/body/div[30]/div[2]/div[2]/div[1]/div') 
elem.date_20180101$clickElement() 
```

Click Botton of Quering Tickets
===============================

``` r
Sys.sleep(1 + runif(1)) 
elem.query_tickets <- remDr$findElement(using = 'xpath', '//*[@id="query_ticket"]') 
elem.query_tickets$clickElement() 
```

Download Tickets Table
======================

``` r
# Scroll to the bottom and wait 3 seconds 
remDr$executeScript('window.scrollTo(0, document.body.scrollHeight);', args = list("dummy")) 
```

    ## list()

``` r
Sys.sleep(1 + runif(1)) 
page_12306_source <- remDr$getPageSource() 
page_12306_source_html <- read_html(page_12306_source[[1]])
```

``` r
tickets_table <- page_12306_source_html %>% 
    html_node('div#t-list.t-list') %>% 
    html_node('table') %>% 
    html_table(fill = TRUE)
```

``` r
#  [\u4e00-\u9fff]+  match any Chinese Characters 

tickets_table.clean <- tickets_table %>% 
    set_names(c('trainNo', 'fromTo', 'timeSchedule', 'during1', 
                'bussinessTickets', 'firstClass', 'secondClass', 
                'superSoftBed', 'softBed', 'crhBed', 'hardBed', 
                'softChair', 'hardChair', 'stand', 'other', 'remarks')) %>% 
    filter(!is.na(trainNo)) %>% 
    mutate(
        trainNo = str_extract(trainNo, "[A-Z]\\d+"), 
        fromTo = sub('查看票价', '', str_extract(fromTo, "查看票价[\u4e00-\u9fff]+")), 
        theDayOrNext =  str_split_fixed(during1, '\\d{2}:\\d{2}', 4)[,4] 
    )  
train_schedule <- str_extract_all(tickets_table.clean$during1, '\\d{2}:\\d{2}', 3) %>% 
    as.data.frame() %>% 
    set_names('start', 'arrival', 'during')

tickets_table.clean <- cbind(tickets_table.clean, train_schedule) %>% 
    select(-timeSchedule, -during1)
knitr::kable(tickets_table.clean) 
```

| trainNo | fromTo       | bussinessTickets | firstClass | secondClass | superSoftBed | softBed | crhBed | hardBed | softChair | hardChair | stand | other | remarks | theDayOrNext | start | arrival | during |
|:--------|:-------------|:-----------------|:-----------|:------------|:-------------|:--------|:-------|:--------|:----------|:----------|:------|:------|:--------|:-------------|:------|:--------|:-------|
| G34     | 杭州东北京南 | 有               | 有         | 有          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 07:10 | 13:05   | 05:55  |
| G20     | 杭州东北京南 | 有               | 有         | 有          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 08:30 | 13:32   | 05:02  |
| G58     | 杭州东北京南 | 8                | 有         | 有          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 08:50 | 14:44   | 05:54  |
| G36     | 杭州东北京南 | 有               | 有         | 19          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 09:05 | 14:53   | 05:48  |
| G42     | 杭州东北京南 | 11               | 有         | 有          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 09:24 | 16:06   | 06:42  |
| G46     | 杭州东北京南 | 7                | 有         | 无          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 09:51 | 15:28   | 05:37  |
| G168    | 杭州东北京南 | 2                | 2          | 无          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 11:36 | 18:03   | 06:27  |
| G38     | 杭州东北京南 | 无               | 无         | 无          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 11:42 | 17:57   | 06:15  |
| G56     | 杭州东北京南 | 无               | 无         | 无          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 12:49 | 18:46   | 05:57  |
| G164    | 杭州东北京南 | 无               | 无         | 无          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 13:41 | 19:54   | 06:13  |
| G40     | 杭州东北京南 | 无               | 5          | 无          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 15:12 | 20:56   | 05:44  |
| G166    | 杭州东北京南 | 4                | 无         | 无          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 15:25 | 21:34   | 06:09  |
| G60     | 杭州东北京南 | 无               | 1          | 无          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 15:43 | 21:29   | 05:46  |
| G44     | 杭州东北京南 | 3                | 有         | 有          | --           | --      | --     | --      | --        | --        | --    | --    | 预订    | 当日到达     | 16:15 | 23:01   | 06:46  |
| Z10     | 杭州北京     | --               | --         | --          | --           | 无      | --     | 无      | --        | 有        | 有    | --    | 预订    | 次日到达     | 17:17 | 07:34   | 14:17  |
| T32     | 杭州北京     | --               | --         | --          | 无           | 无      | --     | 有      | --        | 有        | 有    | 无    | 预订    | 次日到达     | 18:20 | 10:26   | 16:06  |
| K102    | 杭州北京     | --               | --         | --          | --           | 有      | --     | 有      | --        | 有        | 有    | --    | 预订    | 次日到达     | 19:23 | 15:43   | 20:20  |

``` r
remDr$close() 
```
