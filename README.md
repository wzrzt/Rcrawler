# Rcrawler
RSelenium + rvest
  
A few weeks ago, I started to learn scrawling data from website with Python and Selenium module. It is convenient and easy to start. Though it's a long way to go. Still I found fun in it.    
On the other day, I went to ShangHai and attend R Conference in Huadongshifan University. In the conference I learned that there is a package RSelenium which works exactly well in R language just as Selenium in Python. What's more, rvest by Haldey is really a magic tool to parse html. Pipe commands looks very tidy and clear. It also saves my time. Since network speed is the most limitation when crawling with only one session, saving time of coding is very significant.   
   
Here is an example of rvest.   
  
```r 
tickets_table <- page_12306_source_html %>% 
    html_node('div#t-list.t-list') %>% 
    html_node('table') %>% 
    html_table(fill = TRUE)
```
