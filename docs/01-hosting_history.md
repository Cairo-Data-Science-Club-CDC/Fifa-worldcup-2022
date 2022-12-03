

# History of hosting FIFA World Cup

**Overview**

An exploratory data analysis project on the history of hosting FIFA world cup.

**Outline**

1.  [Where and how can we get the data?]

    To start with, we will learn how to scrape Wikipedia directly into R, parse the data tables, and apply quality control to make them ready for the analysis.

    ![](images/aMjwwYeTrEkCyr6af-th-2721308714.jpg){width="265"}

2.  [How many world cups were hosted in each continent?]

    Will then move on to explore the number of hosted cups at the level of continents and the geographical distribution of hosting countries.

    ![](images/unnamed-chunk-25-1.png){width="490"}![](images/unnamed-chunk-16-1.png){width="490"}

3.  [What is the timeline of hosting the world cup?]

    Next, will add the time component by generating a condensed timeline of the history of hosting world cups on the level countries and continents.

    ![](images/unnamed-chunk-33-1.png){width="494"}

4.  [What is the history of bidding for world cup?]

    Finally, we will go beyond the mere hosting the championship to explore the bidding process and the performance of the hosting team over the years.

    ![](images/unnamed-chunk-37-1.png){width="487"}![](images/unnamed-chunk-41-1.png){width="498"}![](images/unnamed-chunk-51-1.png){width="499"}

Let's start by loading the libraries that we'll utilize in our analysis


```r
#web scrapping
library(rvest)
#everything tidy?
library(tidyverse)
#handling spatial-data 
library(rnaturalearth)
library(rnaturalearthdata)
library(sf)
library(ggflags)
library(ggspatial)
library(giscoR)
library(rasterpic)
library(countrycode)
#adding flags in ggplot
library(ggimage)
#
library(visdat)
#set the default ggplot theme
theme_set(cowplot::theme_cowplot())
```

and with that, we're ready to ride!

![](https://media.giphy.com/media/kgsqn9gCVAQ3YM3C2f/giphy.gif)

## Where and how can we get the data?

### Data retrieval

Generally, we would like to know **who** (country, continent) *hosted* **when**. Since *hosting* is a lengthy process that starts by *bidding* and followed by FIFA evaluation. it would be interesting to incorporate *bidding* data into the analysis.

In this project we will use the data made available in this Wikipedia article about [FIFA World Cup hosts](https://en.wikipedia.org/wiki/FIFA_World_Cup_hosts)

To do that, we are going to use the [rvest](https://rvest.tidyverse.org/) package to explore and scrape this tables directly into R.


```r
# URL of the article
url <- "https://en.wikipedia.org/wiki/FIFA_World_Cup_hosts"
# Read the webpage and obtain the pieces of the article containing tables
tbls_lst <-  url %>%
  read_html %>%
  html_table()
#number of retrieved tables
length(tbls_lst)
```

```
## [1] 15
```

We've scrapped the Wikipedia article and parsed all the tables, 15 in total! hmm, we don't need all of them for our analysis. Let's select only the tables of interest for this tutorial. We'll limit our data analysis to the subset of tables showing the list of countries that have submitted a bid or actually hosted the world cup and the performance of host countries in our analysis.


```r
# Select tables of interest
tbls_lst <- tbls_lst[c(1,9,10)]

# Assign names to the tables
tables_names <- c("List of hosts",
                  "Total bids by country",
                  "Host country performances")
names(tbls_lst) <- tolower(tables_names) %>% str_replace_all(" ","_")
```

Let's have a quick look at the top of the selected tables


```r
gt::gt(head(tbls_lst$list_of_hosts))
```

```{=html}
<div id="qkeypcpizj" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#qkeypcpizj .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#qkeypcpizj .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#qkeypcpizj .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#qkeypcpizj .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#qkeypcpizj .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qkeypcpizj .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#qkeypcpizj .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#qkeypcpizj .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#qkeypcpizj .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#qkeypcpizj .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#qkeypcpizj .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#qkeypcpizj .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
}

#qkeypcpizj .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#qkeypcpizj .gt_from_md > :first-child {
  margin-top: 0;
}

#qkeypcpizj .gt_from_md > :last-child {
  margin-bottom: 0;
}

#qkeypcpizj .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#qkeypcpizj .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#qkeypcpizj .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#qkeypcpizj .gt_row_group_first td {
  border-top-width: 2px;
}

#qkeypcpizj .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#qkeypcpizj .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#qkeypcpizj .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#qkeypcpizj .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qkeypcpizj .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#qkeypcpizj .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#qkeypcpizj .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#qkeypcpizj .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qkeypcpizj .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#qkeypcpizj .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-left: 4px;
  padding-right: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#qkeypcpizj .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#qkeypcpizj .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#qkeypcpizj .gt_left {
  text-align: left;
}

#qkeypcpizj .gt_center {
  text-align: center;
}

#qkeypcpizj .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#qkeypcpizj .gt_font_normal {
  font-weight: normal;
}

#qkeypcpizj .gt_font_bold {
  font-weight: bold;
}

#qkeypcpizj .gt_font_italic {
  font-style: italic;
}

#qkeypcpizj .gt_super {
  font-size: 65%;
}

#qkeypcpizj .gt_footnote_marks {
  font-style: italic;
  font-weight: normal;
  font-size: 75%;
  vertical-align: 0.4em;
}

#qkeypcpizj .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#qkeypcpizj .gt_indent_1 {
  text-indent: 5px;
}

#qkeypcpizj .gt_indent_2 {
  text-indent: 10px;
}

#qkeypcpizj .gt_indent_3 {
  text-indent: 15px;
}

#qkeypcpizj .gt_indent_4 {
  text-indent: 20px;
}

#qkeypcpizj .gt_indent_5 {
  text-indent: 25px;
}
</style>
<table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">Year</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col">Host nation(s)</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col">Continent</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td class="gt_row gt_right">1930</td>
<td class="gt_row gt_left">Uruguay</td>
<td class="gt_row gt_left">South America</td></tr>
    <tr><td class="gt_row gt_right">1934</td>
<td class="gt_row gt_left">Italy</td>
<td class="gt_row gt_left">Europe</td></tr>
    <tr><td class="gt_row gt_right">1938</td>
<td class="gt_row gt_left">France</td>
<td class="gt_row gt_left">Europe</td></tr>
    <tr><td class="gt_row gt_right">1942</td>
<td class="gt_row gt_left">Cancelled because of World War II</td>
<td class="gt_row gt_left">Cancelled because of World War II</td></tr>
    <tr><td class="gt_row gt_right">1946</td>
<td class="gt_row gt_left">Cancelled because of World War II</td>
<td class="gt_row gt_left">Cancelled because of World War II</td></tr>
    <tr><td class="gt_row gt_right">1950</td>
<td class="gt_row gt_left">Brazil</td>
<td class="gt_row gt_left">South America</td></tr>
  </tbody>
  
  
</table>
</div>
```

```r
gt::gt(head(tbls_lst$total_bids_by_country))
```

```{=html}
<div id="cjlqqrddsb" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#cjlqqrddsb .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#cjlqqrddsb .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#cjlqqrddsb .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#cjlqqrddsb .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#cjlqqrddsb .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#cjlqqrddsb .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#cjlqqrddsb .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#cjlqqrddsb .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#cjlqqrddsb .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#cjlqqrddsb .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#cjlqqrddsb .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#cjlqqrddsb .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
}

#cjlqqrddsb .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#cjlqqrddsb .gt_from_md > :first-child {
  margin-top: 0;
}

#cjlqqrddsb .gt_from_md > :last-child {
  margin-bottom: 0;
}

#cjlqqrddsb .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#cjlqqrddsb .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#cjlqqrddsb .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#cjlqqrddsb .gt_row_group_first td {
  border-top-width: 2px;
}

#cjlqqrddsb .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#cjlqqrddsb .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#cjlqqrddsb .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#cjlqqrddsb .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#cjlqqrddsb .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#cjlqqrddsb .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#cjlqqrddsb .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#cjlqqrddsb .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#cjlqqrddsb .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#cjlqqrddsb .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-left: 4px;
  padding-right: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#cjlqqrddsb .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#cjlqqrddsb .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#cjlqqrddsb .gt_left {
  text-align: left;
}

#cjlqqrddsb .gt_center {
  text-align: center;
}

#cjlqqrddsb .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#cjlqqrddsb .gt_font_normal {
  font-weight: normal;
}

#cjlqqrddsb .gt_font_bold {
  font-weight: bold;
}

#cjlqqrddsb .gt_font_italic {
  font-style: italic;
}

#cjlqqrddsb .gt_super {
  font-size: 65%;
}

#cjlqqrddsb .gt_footnote_marks {
  font-style: italic;
  font-weight: normal;
  font-size: 75%;
  vertical-align: 0.4em;
}

#cjlqqrddsb .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#cjlqqrddsb .gt_indent_1 {
  text-indent: 5px;
}

#cjlqqrddsb .gt_indent_2 {
  text-indent: 10px;
}

#cjlqqrddsb .gt_indent_3 {
  text-indent: 15px;
}

#cjlqqrddsb .gt_indent_4 {
  text-indent: 20px;
}

#cjlqqrddsb .gt_indent_5 {
  text-indent: 25px;
}
</style>
<table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col">Country</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">Bids</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col">Years</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">Times  hosted</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td class="gt_row gt_left">Germany</td>
<td class="gt_row gt_right">8</td>
<td class="gt_row gt_left">1938, 1962,[a] 1966,[a]1974,[a]1982,[a]1990,[a]1998, 2006</td>
<td class="gt_row gt_right">2</td></tr>
    <tr><td class="gt_row gt_left">Argentina</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_left">1938, 1962, 1970, 1978, 2014</td>
<td class="gt_row gt_right">1</td></tr>
    <tr><td class="gt_row gt_left">England</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_left">1966, 1990, 1998, 2006, 2018</td>
<td class="gt_row gt_right">1</td></tr>
    <tr><td class="gt_row gt_left">Italy</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_left">1930, 1934, 1974, 1982, 1990</td>
<td class="gt_row gt_right">2</td></tr>
    <tr><td class="gt_row gt_left">Mexico</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_left">1970, 1978, 1986,[b] 2002, 2026[c]</td>
<td class="gt_row gt_right">3</td></tr>
    <tr><td class="gt_row gt_left">Morocco</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_left">1994, 1998, 2006, 2010, 2026</td>
<td class="gt_row gt_right">0</td></tr>
  </tbody>
  
  
</table>
</div>
```

```r
gt::gt(head(tbls_lst$host_country_performances))
```

```{=html}
<div id="hunlxfqgai" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#hunlxfqgai .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#hunlxfqgai .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#hunlxfqgai .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#hunlxfqgai .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#hunlxfqgai .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hunlxfqgai .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#hunlxfqgai .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#hunlxfqgai .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#hunlxfqgai .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#hunlxfqgai .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#hunlxfqgai .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#hunlxfqgai .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
}

#hunlxfqgai .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#hunlxfqgai .gt_from_md > :first-child {
  margin-top: 0;
}

#hunlxfqgai .gt_from_md > :last-child {
  margin-bottom: 0;
}

#hunlxfqgai .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#hunlxfqgai .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#hunlxfqgai .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#hunlxfqgai .gt_row_group_first td {
  border-top-width: 2px;
}

#hunlxfqgai .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#hunlxfqgai .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#hunlxfqgai .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#hunlxfqgai .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hunlxfqgai .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#hunlxfqgai .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#hunlxfqgai .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#hunlxfqgai .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#hunlxfqgai .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#hunlxfqgai .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-left: 4px;
  padding-right: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#hunlxfqgai .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#hunlxfqgai .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#hunlxfqgai .gt_left {
  text-align: left;
}

#hunlxfqgai .gt_center {
  text-align: center;
}

#hunlxfqgai .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#hunlxfqgai .gt_font_normal {
  font-weight: normal;
}

#hunlxfqgai .gt_font_bold {
  font-weight: bold;
}

#hunlxfqgai .gt_font_italic {
  font-style: italic;
}

#hunlxfqgai .gt_super {
  font-size: 65%;
}

#hunlxfqgai .gt_footnote_marks {
  font-style: italic;
  font-weight: normal;
  font-size: 75%;
  vertical-align: 0.4em;
}

#hunlxfqgai .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#hunlxfqgai .gt_indent_1 {
  text-indent: 5px;
}

#hunlxfqgai .gt_indent_2 {
  text-indent: 10px;
}

#hunlxfqgai .gt_indent_3 {
  text-indent: 15px;
}

#hunlxfqgai .gt_indent_4 {
  text-indent: 20px;
}

#hunlxfqgai .gt_indent_5 {
  text-indent: 25px;
}
</style>
<table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">Year</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col">Team</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col">Result</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col">Note</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">Pld</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">W</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">D</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">L</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">GF</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">GA</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">GD</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td class="gt_row gt_right">1930</td>
<td class="gt_row gt_left">Uruguay</td>
<td class="gt_row gt_left">Champions</td>
<td class="gt_row gt_left">Best result, later equalled</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">15</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_right">+12</td></tr>
    <tr><td class="gt_row gt_right">1934</td>
<td class="gt_row gt_left">Italy</td>
<td class="gt_row gt_left">Champions</td>
<td class="gt_row gt_left">Best result, later equalled</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">12</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_right">+9</td></tr>
    <tr><td class="gt_row gt_right">1938</td>
<td class="gt_row gt_left">France</td>
<td class="gt_row gt_left">Quarter-finals</td>
<td class="gt_row gt_left">Best result, later improved</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">0</td></tr>
    <tr><td class="gt_row gt_right">1950</td>
<td class="gt_row gt_left">Brazil</td>
<td class="gt_row gt_left">Runners-up</td>
<td class="gt_row gt_left">Best result, later improved</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">22</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_right">+16</td></tr>
    <tr><td class="gt_row gt_right">1954</td>
<td class="gt_row gt_left">Switzerland</td>
<td class="gt_row gt_left">Quarter-finals</td>
<td class="gt_row gt_left">Equalled best result</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">11</td>
<td class="gt_row gt_right">11</td>
<td class="gt_row gt_right">0</td></tr>
    <tr><td class="gt_row gt_right">1958</td>
<td class="gt_row gt_left">Sweden</td>
<td class="gt_row gt_left">Runners-up</td>
<td class="gt_row gt_left">Best result</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">12</td>
<td class="gt_row gt_right">7</td>
<td class="gt_row gt_right">+5</td></tr>
  </tbody>
  
  
</table>
</div>
```

Looks good! Next, we'll have a deeper look at the data to insure that everything is in the right place.

### Data quality control

Parsing data from Web is not a perfect process due to different formatting. We'll start by having a visual inspection of the tables using the package `visdat` and combine all the tables in a single plot.


```r
#Visualize the content of the tables
(vis_dat_lst <- lapply(tbls_lst,visdat::vis_dat))
```

```
## Warning: `gather_()` was deprecated in tidyr 1.2.0.
## ℹ Please use `gather()` instead.
## ℹ The deprecated feature was likely used in the visdat package.
##   Please report the issue at <https://github.com/ropensci/visdat/issues>.
```

```
## $list_of_hosts
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-5-1.png" width="100%" height="100%" />

```
## 
## $total_bids_by_country
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-5-2.png" width="100%" height="100%" />

```
## 
## $host_country_performances
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-5-3.png" width="100%" height="100%" />

```r
#Add title of the data to the plot
(vis_dat_lst <- lapply(names(vis_dat_lst), function(dat_name){
  #subset the plot of by name
  vis <- vis_dat_lst[[dat_name]]
  #add title
  vis + 
    labs(title = dat_name)
  }))
```

```
## [[1]]
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-5-4.png" width="100%" height="100%" />

```
## 
## [[2]]
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-5-5.png" width="100%" height="100%" />

```
## 
## [[3]]
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-5-6.png" width="100%" height="100%" />

```r
#combine all the tables in a single plot
vis_dat_lst %>%
  patchwork::wrap_plots() &
  theme(legend.position = "bottom")
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-5-7.png" width="100%" height="100%" />

The plots reveal two issues. First, the type of the column "Years" in the second table is character! We'll fix this later, but now let's deal with the second issue. The third table shows missing data in many columns! Let's have a deeper look on this table


```r
gt::gt(tbls_lst[[3]])
```

```{=html}
<div id="ippyzmvgog" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#ippyzmvgog .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#ippyzmvgog .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#ippyzmvgog .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#ippyzmvgog .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#ippyzmvgog .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ippyzmvgog .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#ippyzmvgog .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#ippyzmvgog .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#ippyzmvgog .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#ippyzmvgog .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#ippyzmvgog .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#ippyzmvgog .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
}

#ippyzmvgog .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#ippyzmvgog .gt_from_md > :first-child {
  margin-top: 0;
}

#ippyzmvgog .gt_from_md > :last-child {
  margin-bottom: 0;
}

#ippyzmvgog .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#ippyzmvgog .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#ippyzmvgog .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#ippyzmvgog .gt_row_group_first td {
  border-top-width: 2px;
}

#ippyzmvgog .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ippyzmvgog .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#ippyzmvgog .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#ippyzmvgog .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ippyzmvgog .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ippyzmvgog .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#ippyzmvgog .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#ippyzmvgog .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ippyzmvgog .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#ippyzmvgog .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-left: 4px;
  padding-right: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#ippyzmvgog .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#ippyzmvgog .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#ippyzmvgog .gt_left {
  text-align: left;
}

#ippyzmvgog .gt_center {
  text-align: center;
}

#ippyzmvgog .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#ippyzmvgog .gt_font_normal {
  font-weight: normal;
}

#ippyzmvgog .gt_font_bold {
  font-weight: bold;
}

#ippyzmvgog .gt_font_italic {
  font-style: italic;
}

#ippyzmvgog .gt_super {
  font-size: 65%;
}

#ippyzmvgog .gt_footnote_marks {
  font-style: italic;
  font-weight: normal;
  font-size: 75%;
  vertical-align: 0.4em;
}

#ippyzmvgog .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#ippyzmvgog .gt_indent_1 {
  text-indent: 5px;
}

#ippyzmvgog .gt_indent_2 {
  text-indent: 10px;
}

#ippyzmvgog .gt_indent_3 {
  text-indent: 15px;
}

#ippyzmvgog .gt_indent_4 {
  text-indent: 20px;
}

#ippyzmvgog .gt_indent_5 {
  text-indent: 25px;
}
</style>
<table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">Year</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col">Team</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col">Result</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col">Note</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">Pld</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">W</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">D</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">L</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">GF</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">GA</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1" scope="col">GD</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td class="gt_row gt_right">1930</td>
<td class="gt_row gt_left">Uruguay</td>
<td class="gt_row gt_left">Champions</td>
<td class="gt_row gt_left">Best result, later equalled</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">15</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_left">+12</td></tr>
    <tr><td class="gt_row gt_right">1934</td>
<td class="gt_row gt_left">Italy</td>
<td class="gt_row gt_left">Champions</td>
<td class="gt_row gt_left">Best result, later equalled</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">12</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_left">+9</td></tr>
    <tr><td class="gt_row gt_right">1938</td>
<td class="gt_row gt_left">France</td>
<td class="gt_row gt_left">Quarter-finals</td>
<td class="gt_row gt_left">Best result, later improved</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_left">0</td></tr>
    <tr><td class="gt_row gt_right">1950</td>
<td class="gt_row gt_left">Brazil</td>
<td class="gt_row gt_left">Runners-up</td>
<td class="gt_row gt_left">Best result, later improved</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">22</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_left">+16</td></tr>
    <tr><td class="gt_row gt_right">1954</td>
<td class="gt_row gt_left">Switzerland</td>
<td class="gt_row gt_left">Quarter-finals</td>
<td class="gt_row gt_left">Equalled best result</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">11</td>
<td class="gt_row gt_right">11</td>
<td class="gt_row gt_left">0</td></tr>
    <tr><td class="gt_row gt_right">1958</td>
<td class="gt_row gt_left">Sweden</td>
<td class="gt_row gt_left">Runners-up</td>
<td class="gt_row gt_left">Best result</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">12</td>
<td class="gt_row gt_right">7</td>
<td class="gt_row gt_left">+5</td></tr>
    <tr><td class="gt_row gt_right">1962</td>
<td class="gt_row gt_left">Chile</td>
<td class="gt_row gt_left">Third place</td>
<td class="gt_row gt_left">Best result</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">10</td>
<td class="gt_row gt_right">8</td>
<td class="gt_row gt_left">+2</td></tr>
    <tr><td class="gt_row gt_right">1966</td>
<td class="gt_row gt_left">England</td>
<td class="gt_row gt_left">Champions</td>
<td class="gt_row gt_left">Best result</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">11</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_left">+8</td></tr>
    <tr><td class="gt_row gt_right">1970</td>
<td class="gt_row gt_left">Mexico</td>
<td class="gt_row gt_left">Quarter-finals</td>
<td class="gt_row gt_left">Best result, later equalled (again as hosts)</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_left">+2</td></tr>
    <tr><td class="gt_row gt_right">1974</td>
<td class="gt_row gt_left">West Germany</td>
<td class="gt_row gt_left">Champions</td>
<td class="gt_row gt_left">Equalled best result, later equalled again</td>
<td class="gt_row gt_right">7</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">13</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_left">+9</td></tr>
    <tr><td class="gt_row gt_right">1978</td>
<td class="gt_row gt_left">Argentina</td>
<td class="gt_row gt_left">Champions</td>
<td class="gt_row gt_left">Best result, later equalled</td>
<td class="gt_row gt_right">7</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">15</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_left">+9</td></tr>
    <tr><td class="gt_row gt_right">1982</td>
<td class="gt_row gt_left">Spain</td>
<td class="gt_row gt_left">Second round (top 12)</td>
<td class="gt_row gt_left"></td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_left">−1</td></tr>
    <tr><td class="gt_row gt_right">1986</td>
<td class="gt_row gt_left">Mexico</td>
<td class="gt_row gt_left">Quarter-finals</td>
<td class="gt_row gt_left">Equalled best result (previous time again as hosts)</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_left">+4</td></tr>
    <tr><td class="gt_row gt_right">1990</td>
<td class="gt_row gt_left">Italy</td>
<td class="gt_row gt_left">Third place</td>
<td class="gt_row gt_left"></td>
<td class="gt_row gt_right">7</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">10</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_left">+8</td></tr>
    <tr><td class="gt_row gt_right">1994</td>
<td class="gt_row gt_left">United States</td>
<td class="gt_row gt_left">Round of 16</td>
<td class="gt_row gt_left"></td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_left">−1</td></tr>
    <tr><td class="gt_row gt_right">1998</td>
<td class="gt_row gt_left">France</td>
<td class="gt_row gt_left">Champions</td>
<td class="gt_row gt_left">Best result, later equalled</td>
<td class="gt_row gt_right">7</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">15</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_left">+13</td></tr>
    <tr><td class="gt_row gt_right">2002</td>
<td class="gt_row gt_left">South Korea</td>
<td class="gt_row gt_left">Fourth place</td>
<td class="gt_row gt_left">Best result</td>
<td class="gt_row gt_right">7</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">8</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_left">+2</td></tr>
    <tr><td class="gt_row gt_right">2002</td>
<td class="gt_row gt_left">Japan</td>
<td class="gt_row gt_left">Round of 16</td>
<td class="gt_row gt_left">Best result, later equalled</td>
<td class="gt_row gt_right">4</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_left">+2</td></tr>
    <tr><td class="gt_row gt_right">2006</td>
<td class="gt_row gt_left">Germany</td>
<td class="gt_row gt_left">Third place</td>
<td class="gt_row gt_left"></td>
<td class="gt_row gt_right">7</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">14</td>
<td class="gt_row gt_right">6</td>
<td class="gt_row gt_left">+8</td></tr>
    <tr><td class="gt_row gt_right">2010</td>
<td class="gt_row gt_left">South Africa</td>
<td class="gt_row gt_left">First round</td>
<td class="gt_row gt_left">Equalled best (and worst) result</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_left">−2</td></tr>
    <tr><td class="gt_row gt_right">2014</td>
<td class="gt_row gt_left">Brazil</td>
<td class="gt_row gt_left">Fourth place</td>
<td class="gt_row gt_left"></td>
<td class="gt_row gt_right">7</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">11</td>
<td class="gt_row gt_right">14</td>
<td class="gt_row gt_left">−3</td></tr>
    <tr><td class="gt_row gt_right">2018</td>
<td class="gt_row gt_left">Russia</td>
<td class="gt_row gt_left">Quarter-finals</td>
<td class="gt_row gt_left">Best result as independent nation</td>
<td class="gt_row gt_right">5</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">2</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">11</td>
<td class="gt_row gt_right">7</td>
<td class="gt_row gt_left">+4</td></tr>
    <tr><td class="gt_row gt_right">2022</td>
<td class="gt_row gt_left">Qatar</td>
<td class="gt_row gt_left">First round</td>
<td class="gt_row gt_left">Best (and worst) result</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">3</td>
<td class="gt_row gt_right">1</td>
<td class="gt_row gt_right">7</td>
<td class="gt_row gt_left">-6</td></tr>
    <tr><td class="gt_row gt_right">2026</td>
<td class="gt_row gt_left">Canada</td>
<td class="gt_row gt_left">TBD</td>
<td class="gt_row gt_left"></td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_left"></td></tr>
    <tr><td class="gt_row gt_right">2026</td>
<td class="gt_row gt_left">Mexico</td>
<td class="gt_row gt_left">TBD</td>
<td class="gt_row gt_left"></td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_left"></td></tr>
    <tr><td class="gt_row gt_right">2026</td>
<td class="gt_row gt_left">United States</td>
<td class="gt_row gt_left">TBD</td>
<td class="gt_row gt_left"></td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_right">NA</td>
<td class="gt_row gt_left"></td></tr>
  </tbody>
  
  
</table>
</div>
```

Apparently, the missing values are yet-to-be-determined performances of the hosting countries in 2026! Therefore, this shouldn't be a concern and is not a failure of parsing the table.

Before rushing to the analysis, let's push the tables through a few rounds of quality control.

let's start by cleaning column names by handling special characters, spaces, and applying a consistent format.


```r
#names of columns before cleaning
lapply(tbls_lst, colnames)
```

```
## $list_of_hosts
## [1] "Year"           "Host nation(s)" "Continent"     
## 
## $total_bids_by_country
## [1] "Country"       "Bids"          "Years"         "Times  hosted"
## 
## $host_country_performances
##  [1] "Year"   "Team"   "Result" "Note"   "Pld"    "W"      "D"      "L"     
##  [9] "GF"     "GA"     "GD"
```

```r
# Clean columns' names
tbls_lst <- lapply(tbls_lst,  janitor::clean_names)
#names of columns after cleaning
lapply(tbls_lst, colnames)
```

```
## $list_of_hosts
## [1] "year"          "host_nation_s" "continent"    
## 
## $total_bids_by_country
## [1] "country"      "bids"         "years"        "times_hosted"
## 
## $host_country_performances
##  [1] "year"   "team"   "result" "note"   "pld"    "w"      "d"      "l"     
##  [9] "gf"     "ga"     "gd"
```

One can see that the column with the countries has a different name ("host_nation_s", "country", "team") in each table. Let's fix this inconsistency and set an new name ( "country_name") to all of them.


```r
#old inconsistent names
cols_old <- c("country", "team", "host_nation_s")
#new column name
col_new <- "country_name"
#apply the replacement
(tbls_lst <- lapply(tbls_lst, function(tbl){
  tbl %>%
    rename_with(~ ifelse(.x %in% cols_old,
                         col_new,
                         .x))
  }))
```

```
## $list_of_hosts
## # A tibble: 25 × 3
##     year country_name                      continent                        
##    <int> <chr>                             <chr>                            
##  1  1930 Uruguay                           South America                    
##  2  1934 Italy                             Europe                           
##  3  1938 France                            Europe                           
##  4  1942 Cancelled because of World War II Cancelled because of World War II
##  5  1946 Cancelled because of World War II Cancelled because of World War II
##  6  1950 Brazil                            South America                    
##  7  1954 Switzerland                       Europe                           
##  8  1958 Sweden                            Europe                           
##  9  1962 Chile                             South America                    
## 10  1966 England                           Europe                           
## # … with 15 more rows
## 
## $total_bids_by_country
## # A tibble: 35 × 4
##    country_name   bids years                                             times…¹
##    <chr>         <int> <chr>                                               <int>
##  1 Germany           8 1938, 1962,[a] 1966,[a]1974,[a]1982,[a]1990,[a]1…       2
##  2 Argentina         5 1938, 1962, 1970, 1978, 2014                            1
##  3 England           5 1966, 1990, 1998, 2006, 2018                            1
##  4 Italy             5 1930, 1934, 1974, 1982, 1990                            2
##  5 Mexico            5 1970, 1978, 1986,[b] 2002, 2026[c]                      3
##  6 Morocco           5 1994, 1998, 2006, 2010, 2026                            0
##  7 Spain             5 1930, 1966, 1974, 1982, 2018[d]                         1
##  8 Brazil            4 1950, 1994, 2006, 2014                                  2
##  9 Colombia          4 1970, 1978, 1986,[b]2014                                1
## 10 United States     4 1986, 1994, 2022, 2026[c]                               2
## # … with 25 more rows, and abbreviated variable name ¹​times_hosted
## 
## $host_country_performances
## # A tibble: 26 × 11
##     year country_name result     note    pld     w     d     l    gf    ga gd   
##    <int> <chr>        <chr>      <chr> <int> <int> <int> <int> <int> <int> <chr>
##  1  1930 Uruguay      Champions  Best…     4     4     0     0    15     3 +12  
##  2  1934 Italy        Champions  Best…     5     4     1     0    12     3 +9   
##  3  1938 France       Quarter-f… Best…     2     1     0     1     4     4 0    
##  4  1950 Brazil       Runners-up Best…     6     4     1     1    22     6 +16  
##  5  1954 Switzerland  Quarter-f… Equa…     4     2     0     2    11    11 0    
##  6  1958 Sweden       Runners-up Best…     6     4     1     1    12     7 +5   
##  7  1962 Chile        Third pla… Best…     6     4     0     2    10     8 +2   
##  8  1966 England      Champions  Best…     6     5     1     0    11     3 +8   
##  9  1970 Mexico       Quarter-f… Best…     4     2     1     1     6     4 +2   
## 10  1974 West Germany Champions  Equa…     7     6     0     1    13     4 +9   
## # … with 16 more rows
```

Similarly, the year column is called "years" in the second table. Let's make it consistent with the other tables and rename it to "year".


```r
tbls_lst$total_bids_by_country <- tbls_lst$total_bids_by_country %>% 
  dplyr::rename(year = "years")
```

Next, we need to insure that the data is "tidy". Obviously, this is not the case for the table below where the column column "years" show mutliple dates concatenated in the same row


```r
tbls_lst$total_bids_by_country
```

```
## # A tibble: 35 × 4
##    country_name   bids year                                              times…¹
##    <chr>         <int> <chr>                                               <int>
##  1 Germany           8 1938, 1962,[a] 1966,[a]1974,[a]1982,[a]1990,[a]1…       2
##  2 Argentina         5 1938, 1962, 1970, 1978, 2014                            1
##  3 England           5 1966, 1990, 1998, 2006, 2018                            1
##  4 Italy             5 1930, 1934, 1974, 1982, 1990                            2
##  5 Mexico            5 1970, 1978, 1986,[b] 2002, 2026[c]                      3
##  6 Morocco           5 1994, 1998, 2006, 2010, 2026                            0
##  7 Spain             5 1930, 1966, 1974, 1982, 2018[d]                         1
##  8 Brazil            4 1950, 1994, 2006, 2014                                  2
##  9 Colombia          4 1970, 1978, 1986,[b]2014                                1
## 10 United States     4 1986, 1994, 2022, 2026[c]                               2
## # … with 25 more rows, and abbreviated variable name ¹​times_hosted
```

what we need to do is to split years of bids into separate entries and convert it to numeric


```r
#separate concatenated years into separate rows
(tbls_lst$total_bids_by_country <- tbls_lst$total_bids_by_country %>% 
  mutate(year = str_extract_all(year, "[0-9]+")) %>% 
  unnest(year) %>% 
  mutate(year = as.numeric(year)))
```

```
## # A tibble: 90 × 4
##    country_name  bids  year times_hosted
##    <chr>        <int> <dbl>        <int>
##  1 Germany          8  1938            2
##  2 Germany          8  1962            2
##  3 Germany          8  1966            2
##  4 Germany          8  1974            2
##  5 Germany          8  1982            2
##  6 Germany          8  1990            2
##  7 Germany          8  1998            2
##  8 Germany          8  2006            2
##  9 Argentina        5  1938            1
## 10 Argentina        5  1962            1
## # … with 80 more rows
```

and do the same thing by splitting cohosts of the same world cup (e.g. "Japan South Korea") into separate rows entries ("Japan", "South Korea").


```r
#separate cohosting countries into separate entries
tbls_lst$list_of_hosts <- tbls_lst$list_of_hosts %>% 
  mutate(country_name = str_split(country_name, "\\s{2}")) %>%
  unnest(country_name) %>% 
  dplyr::rename(host_year = "year")
```

Have a look on `separate_rows()` for another way to achieve the same effect

Next, let's give a meaningful order to the results of the teams.


```r
#order of the results
results_order <- c("Champions",
                   "Runners-up",
                   "Third place",
                   "Fourth place",
                   "Quarter-finals",
                   "Round of 16",
                   "Second round",
                   "First round",
                   "TBD"
                   )
#set the order
tbls_lst$host_country_performances  <- tbls_lst$host_country_performances %>%
  mutate(result = ifelse(result == "Second round (top 12)", "Second round", result),
         result = factor(result, levels = results_order))
```

And we'll end this part by defining a new column with country code (isoc2).


```r
#get iso2 code of each country (flags for Yugoslavia and England are missing)
(tbls_lst <- lapply(tbls_lst,function(tbl){
  tbl$country_code <- countrycode::countrycode(tbl$country_name,
                                            "country.name",# the provided country label
                                            "iso2c"# the country code
                                            )
  tbl
  }))
```

```
## Warning in countrycode_convert(sourcevar = sourcevar, origin = origin, destination = dest, : Some values were not matched unambiguously: Cancelled because of World War II, England
```

```
## Warning in countrycode_convert(sourcevar = sourcevar, origin = origin, destination = dest, : Some values were not matched unambiguously: England, Yugoslavia
```

```
## Warning in countrycode_convert(sourcevar = sourcevar, origin = origin, destination = dest, : Some values were not matched unambiguously: England
```

```
## $list_of_hosts
## # A tibble: 28 × 4
##    host_year country_name                      continent                 count…¹
##        <int> <chr>                             <chr>                     <chr>  
##  1      1930 Uruguay                           South America             UY     
##  2      1934 Italy                             Europe                    IT     
##  3      1938 France                            Europe                    FR     
##  4      1942 Cancelled because of World War II Cancelled because of Wor… <NA>   
##  5      1946 Cancelled because of World War II Cancelled because of Wor… <NA>   
##  6      1950 Brazil                            South America             BR     
##  7      1954 Switzerland                       Europe                    CH     
##  8      1958 Sweden                            Europe                    SE     
##  9      1962 Chile                             South America             CL     
## 10      1966 England                           Europe                    <NA>   
## # … with 18 more rows, and abbreviated variable name ¹​country_code
## 
## $total_bids_by_country
## # A tibble: 90 × 5
##    country_name  bids  year times_hosted country_code
##    <chr>        <int> <dbl>        <int> <chr>       
##  1 Germany          8  1938            2 DE          
##  2 Germany          8  1962            2 DE          
##  3 Germany          8  1966            2 DE          
##  4 Germany          8  1974            2 DE          
##  5 Germany          8  1982            2 DE          
##  6 Germany          8  1990            2 DE          
##  7 Germany          8  1998            2 DE          
##  8 Germany          8  2006            2 DE          
##  9 Argentina        5  1938            1 AR          
## 10 Argentina        5  1962            1 AR          
## # … with 80 more rows
## 
## $host_country_performances
## # A tibble: 26 × 12
##     year countr…¹ result note    pld     w     d     l    gf    ga gd    count…²
##    <int> <chr>    <fct>  <chr> <int> <int> <int> <int> <int> <int> <chr> <chr>  
##  1  1930 Uruguay  Champ… Best…     4     4     0     0    15     3 +12   UY     
##  2  1934 Italy    Champ… Best…     5     4     1     0    12     3 +9    IT     
##  3  1938 France   Quart… Best…     2     1     0     1     4     4 0     FR     
##  4  1950 Brazil   Runne… Best…     6     4     1     1    22     6 +16   BR     
##  5  1954 Switzer… Quart… Equa…     4     2     0     2    11    11 0     CH     
##  6  1958 Sweden   Runne… Best…     6     4     1     1    12     7 +5    SE     
##  7  1962 Chile    Third… Best…     6     4     0     2    10     8 +2    CL     
##  8  1966 England  Champ… Best…     6     5     1     0    11     3 +8    <NA>   
##  9  1970 Mexico   Quart… Best…     4     2     1     1     6     4 +2    MX     
## 10  1974 West Ge… Champ… Equa…     7     6     0     1    13     4 +9    DE     
## # … with 16 more rows, and abbreviated variable names ¹​country_name,
## #   ²​country_code
```

Quality control is not over yet! We need to manually apply some historical modification to the data.

First, let's correct the entry of Colombia. After being chosen as a host in 1986, the country had to withdrew from hosting the cup due to economic concerns.


```r
tbls_lst$total_bids_by_country <- tbls_lst$total_bids_by_country %>% 
  mutate(times_hosted = ifelse(country_name == "Colombia", 0, times_hosted))
```

Second, as Berlin Wall was brought to the ground few decades ago, let's replace West Germany" with "Germany".

![West Germany national team](https://fifanews.b-cdn.net/wp-content/uploads/2018/02/1974-World-Cup-final-winner-Squad.jpg)


```r
tbls_lst <- lapply(tbls_lst, function(tbl){
  tbl %>%
    mutate(across(where(is.character), #select character columns
                  ~ str_replace(.x, "West Germany", "Germany") #define replacement
                  )
           )
  })
```

Now that the data is analysis-ready, it is time to explore some interesting questions!

![](https://media.giphy.com/media/WmzhEJZsON3Bk7ulRY/giphy.gif)

## How many world cups were hosted in each continent?

Before embarking on our colorful journey of data visualization, let's define a caption that credits the source of the data and the analysis.


```r
caption_cdc <- glue::glue("Data source: {url}\n@Cairo Data Science Club")
theme_update(plot.caption = element_text(face = "italic"))
```

We will exclude the dates in which the championship was cancelled because of Warld War II


```r
df_host1 <- tbls_lst$list_of_hosts %>% 
  filter(!str_detect(continent, "Cancelled")) %>%
  group_by(host_year) %>%
  summarise(across(everything(),
                   ~ paste(unique(.x),collapse = ", ")
                   )
            ) %>%
  ungroup() %>% 
  as.data.frame()
```

Let's look at a basic plot of the data


```r
(bar_host_plt <- df_host1 %>% 
  ggplot(aes(continent))+
  geom_bar())
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-19-1.png" width="100%" height="100%" />

This doesn't look pretty. Let's make it more attractive! First, let's add a some colors.


```r
#Assign colors to each continent
conti_cols <- c(Europe = "#1f78b4",
                Asia = "#6a3d9a",
                `South America` = "#ffff99",
                `North America` = "#33a02c",
                Africa = "#ff7f00")
#show colors in the plot
(bar_host_plt <- bar_host_plt +
    geom_bar(aes(fill = continent))+
  scale_color_manual(values = conti_cols)+
  scale_fill_manual(values = conti_cols))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-20-1.png" width="100%" height="100%" />

Second, let's add some text and remove the legend since it doesn't add to the plot.


```r
(bar_host_plt <- bar_host_plt +
    labs(title = "History of hosting world cup",
       subtitle = "Number of hosted world cups and hosting countries per continents",
       caption = caption_cdc))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-21-1.png" width="100%" height="100%" />

Next, I think we can get rid off the axis and label each bar with important information.


```r
df_host2 <- df_host1 %>% 
  group_by(continent) %>% 
  summarise(n = n())%>%
  ungroup() %>% 
  arrange(n) %>%
  mutate(continent = factor(continent, levels = unique(continent)))

df_host_3 <- df_host2 %>% 
              mutate(cont_n = glue::glue("{continent}\n(n = {n})"))
(bar_host_plt <- df_host2 %>% 
  ggplot(aes(continent, n))+
  geom_col(aes(color = continent), fill = "white",  show.legend = FALSE, linewidth = 1)+
  geom_col(aes(fill = continent), alpha = 0.4, show.legend = FALSE)+
  geom_text(data = df_host_3,
            aes(label = cont_n), size = 5,nudge_y = 0.6)+
  labs(title = "History of hosting world cup",
       subtitle = "Number of hosted world cups and hosting countries per continents",
       caption = caption_cdc)+
  scale_color_manual(values = conti_cols)+
  scale_fill_manual(values = conti_cols)+
  theme(axis.line = element_blank(),
        axis.ticks = element_blank(),
        axis.text = element_blank(),
        axis.title = element_blank()))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-22-1.png" width="100%" height="100%" />

These long bars, there's something about them. Something ... empty. I got an idea! Why not fill them with the names of the hosting countries?! That would look awesome!

The goal is stack countries' names in an equidistant fashion within each bar. To do this, we divide the bar, which indicates the number of hosted world cups, by the number of hosting countries. This would simply partition the bar into equally-sized parts equal to the number of hosting countries. Finally, to get the location of countries within the bar, we compute the cumulative sum of the size of these parts.


```r
df_txt_bar <- df_host1 %>%
              group_by(continent, country_name) %>% 
              summarise(n_host = n()) %>% # number of hosted world cups per country
              group_by(continent) %>% 
              mutate(n_countries = n(), #number of hosting countries 
                     n_cont = sum(n_host), #number of hosted world cups per continent (height of the bar)
                     prop = n_cont/(n_countries+1), # divides the bar height by the number of countries
                     cum_prop = cumsum(prop) #stack countries' names within the bar
                     )%>%
              ungroup() %>% 
              mutate(country_name = ifelse(n_host >1 , glue::glue("{country_name} x {n_host}"), country_name) # indicate the number of times a country hosted the world cup
                     )
```

```
## `summarise()` has grouped output by 'continent'. You can override using the
## `.groups` argument.
```

Let's fill these empty bars with the names of the hosting countries


```r
bar_host_plt <- bar_host_plt +
    geom_text(data = df_txt_bar,
            aes(y = cum_prop, label = country_name),
            size = 4)
```

This worked nicely! I will finalize this plot by adding an image of world cup in the background! Yes, you can do this in R. I need to mention that I got the code for the function `trasparent` below from an answer [here](#https://stackoverflow.com/questions/60820565/is-there-a-way-to-add-an-alpha-value-within-geom-image-in-ggplot).


```r
#get image
wc_img <- here::here("images/wc12.png")
#define function to control transparency and set to 0.2

transparent <- function(img) {
  magick::image_fx(img, expression = "0.2*a", channel = "alpha")
}
#add the image in the background
bar_host_plt+
  ggimage::geom_image(data = data.frame(x = 1.5, y = 7.5),
                      aes(x,y),
                      image = wc_img,image_fun = transparent,
                      size = 1)
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-25-1.png" width="100%" height="100%" />

Isn't that nice?!

![](https://media.giphy.com/media/UQsYRkg6DLbJ6eIrYV/giphy.gif)

Let's add some geographical-context to these results by throwing the data on the world map and see how it would look like.


```r
#get map of the world
world <- ne_countries(scale = "medium", returnclass = "sf")
#and map of separate host countries
host_iso2 <- na.omit(unique(tbls_lst$list_of_hosts$country_code))
wcp_hosts <- gisco_get_countries(country = host_iso2,
                                 epsg = 3857# Pseudo-Mercator projection
                                 )
wcp_hosts$iso2 <- host_iso2
```

Plotting base map of the world using ggplot.


```r
# Base map of the world
(plot <- ggplot(world) +
  geom_sf(fill = "grey90") +
  theme_minimal() +
  theme(panel.background = element_rect(fill = "lightblue")))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-27-1.png" width="100%" height="100%" />

Additionally, let's make the map extra flashy by filling hosting countries with their maps.


```r
# get flags form this repo
flagrepo <- "https://raw.githubusercontent.com/hjnilsson/country-flags/master/png250px/"
```

Finally, we'll download the flags


```r
# Loop and add
for (iso in host_iso2) {
  # Download pic and plot
  imgurl <- paste0(flagrepo, tolower(iso), ".png")
  tmpfile <- tempfile(fileext = ".png")
  download.file(imgurl, tmpfile, quiet = TRUE, mode = "wb")
  
  # Raster
  x <- wcp_hosts %>% filter(iso2 == iso)
  x_rast <- rasterpic_img(x, tmpfile, crop = TRUE, mask = TRUE)
  plot <- plot + layer_spatial(x_rast)
}
```

and add them to world map


```r
plot +
  geom_sf(data = wcp_hosts, fill = NA)+
  labs(title = "World map of FIFA world cup hosts")
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-30-1.png" width="100%" height="100%" />

I'm very happy with the end result!

![](https://media.giphy.com/media/jdBJkDYMphKaf89OTl/giphy.gif)\

Have a look at this [excellent blog](https://dieghernan.github.io/202201_maps-flags/) for more details on adding flags to maps This is where I got to know and learn this trick.

## What is the timeline of hosting the world cup?

What was missing from the previous representation of the data is the time component. In this section we'll explore a visualization method that would allow us to add this crucial aspect.

Let's start by preparing the hosting data for visualization by filling in the gap years where the world cup stopped due to WWII


```r
#add missing years in which world cup was cancelled
df_tm <- df_host1 %>% 
  complete(host_year = full_seq(host_year, 4)) %>% 
  mutate(continent = ifelse(is.na(continent), "Cancelled", continent),
         country_name = ifelse(is.na(country_name), "Cancelled", country_name))
#make a 6x5 grid from the hosting data and add the coordinate of each cell in the grid
df_tm <- df_tm[1:30,] %>% 
  mutate(y = rep(6:1, each = 5),
         x = rep(1:5, 6),
         country_name = case_when(
           str_detect(country_name, "Canada") ~ "Canada\nMexico\nUnited States", #add a new line between cohosts
           str_detect(country_name, "Japan") ~ "Japan\nSouth Korea", #add new line between cohosts
           TRUE ~ country_name),
         continent = factor(continent, levels = unique(continent))) %>% 
  filter(!is.na(continent))
```

Let's use chronologically ordered tiles (AKA waffle plot) to look at the timeline of hosting the world cup.


```r
df_tm %>% 
  ggplot(aes(x, y, fill = continent ))+
  geom_tile(color = "black", size = 1)+
  geom_text(aes(label = country_name), size = 4.5)+
  scale_color_manual(values = conti_cols)+
  scale_fill_manual(values = conti_cols)
```

```
## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
## ℹ Please use `linewidth` instead.
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-32-1.png" width="100%" height="100%" />

That's a good start! Let's add the year information and further beautify the plot.


```r
df_tm %>% 
  ggplot(aes(x, y, fill = continent ))+
  geom_tile(color = "black", size = 1)+
  geom_text(aes(label = country_name), size = 3.1)+
  #add the time interval of each row
  geom_text(data = . %>% 
              group_by(y) %>% 
              mutate(range = glue::glue("({min(host_year)}-{max(host_year)})")) %>% 
              ungroup(),
            aes(label = range),
            x = -0.5,
            size = 3)+
  scale_x_discrete(expand = expansion(add = 2))+
  guides(fill = guide_legend(nrow = 1))+
  labs(title = "History of hosting FIFA world cup",
       subtitle = "Host countries of world cups chronologically orderd and colored by continent",
       caption = caption_cdc)+
  coord_fixed(0.7)+
  scale_color_manual(values = conti_cols)+
  scale_fill_manual(values = conti_cols)+
  theme(title = element_text(size = 10),
        axis.line = element_blank(),
        axis.ticks = element_blank(),
        axis.text = element_blank(),
        axis.title = element_blank(),
        legend.position = "top",
        panel.border = element_rect(linewidth = 2,linetype = "solid", color = "black"))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-33-1.png" width="100%" height="100%" />

This is a condensed and clear representation of our data, which are two desirable features of data visualization.

![](https://media.giphy.com/media/TgL7lPFfyAFITuszOA/giphy.gif)

## What is the history of bidding for world cup?

Now let us shift our focus to yet another interesting question. In this section we will explore the bidding history to host the world cup.

Let's start by making a bar-plot that show the number of successful bids for each country of all submitted bids.

First, sort the countries based on the number of bids, then the times hosted.


```r
(bar_bid_df1 <- tbls_lst$total_bids_by_country %>% 
   distinct(country_name, bids, times_hosted) %>%
  arrange(bids,times_hosted) %>% 
  mutate(country_name = factor(country_name, levels = unique(country_name))) )
```

```
## # A tibble: 35 × 3
##    country_name  bids times_hosted
##    <fct>        <int>        <dbl>
##  1 Austria          1            0
##  2 Belgium          1            0
##  3 Egypt            1            0
##  4 Greece           1            0
##  5 Hungary          1            0
##  6 Iran             1            0
##  7 Libya            1            0
##  8 Nigeria          1            0
##  9 Peru             1            0
## 10 Portugal         1            0
## # … with 25 more rows
```

Next, add a layer of bars showing the times of bids using a transparent color


```r
(bar_bid_plt <- bar_bid_df1 %>% 
  ggplot()+
  geom_col(aes(country_name, bids), fill = "#35978f", alpha = 0.3))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-35-1.png" width="100%" height="100%" />

Then, add yet another layer of bars showing the times hosted using solid version of the same color


```r
(bar_bid_plt <-bar_bid_plt +
  geom_col(aes(country_name, times_hosted), fill = "#35978f", alpha = 1)+
  coord_flip())
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-36-1.png" width="100%" height="100%" />

Finally, add world cup image in the background, text, and customize the theme


```r
(bar_bid_plt +
   #add world cup image
  ggimage::geom_image(data = data.frame(x = 16, y = 7),
                      aes(x,y),
                      image = wc_img,image_fun = transparent,
                      size = 1.1)+
   #add text
  labs(title = "History of hosting FIFA world cup",
     subtitle = "Number of world cup bids compared to times hosted",
     caption = caption_cdc,
       y = "Numer of bids")+
   #define theme
  theme(axis.title.y = element_blank(),
      axis.text.y = element_text(size = 7.6),
      plot.background = element_rect(fill =  "#FAECD6"),
        panel.background = element_rect(fill =  "#FAECD6"),
        title = element_text(colour = "#01665e", size = 9)))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-37-1.png" width="100%" height="100%" />

Nice! The plot shows that most of the time, it takes more than one bid to host the world cup.

![](https://media.giphy.com/media/mxCK3EcADG1F7krLP6/giphy.gif)

Let's enforce the relationship between bids and times hosted using a point plot.

First, let's plot the number of bids on the x-axis and the time hosted on the y-axis


```r
(point_bid_plt <- bar_bid_df1 %>% 
  ggplot(aes(bids, times_hosted))+
  geom_point(color = "#35978f"))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-38-1.png" width="100%" height="100%" />

Next, let's add the name of the country to the repreresentitive point


```r
(point_bid_plt <- point_bid_plt+
  geom_text(aes(label = country_name),
            size = 4))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-39-1.png" width="100%" height="100%" />

Oh no! Since many countries share the same bids and hosting statitics, we end up with a dramatic case of text over-plotting.

![](https://i.gifer.com/3esj.gif)

To overcome this, we'll replace `geom_text()` with `geom_text_repel()` from the package `ggrepel`. Let's first look at the effect of this function and then explain what it does.


```r
#remove the last layer added of geom_text() before using geom_text_repel()
point_bid_plt$layers[[2]] <- NULL
(point_bid_plt <- point_bid_plt+
  ggrepel::geom_text_repel(aes(label = country_name),
                           size = 4,
                           min.segment.length = 0,
                           max.overlaps = Inf,
                           segment.color="grey60",
                           box.padding = 0.4
                            )+
  #expand the plotting panel to free some room for the repelled text
  scale_x_continuous(breaks = 1:8,
                     expand = expansion(add = c(1,0.5)))+
  scale_y_continuous(breaks = 0:3,
                     expand = expansion(add = c(1,0.5))))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-40-1.png" width="100%" height="100%" />

As the name says, `geom_text_repel()` makes the text repel away from each other to avoid over-plotting. The text also repel away from the edges of the plot. To avoid the undesired effect of later, we expanded the plotting are using the function `expansion()` in x and y direction

Finally, add world cup image, the title, and beautify the plot by coloring the background


```r
point_bid_plt+
   #add world cup image
  ggimage::geom_image(data = data.frame(x = 7, y = 1.2),
                      aes(x,y),
                      image = wc_img,image_fun = transparent,
                      size = 1.2)+
  labs(title = "History of hosting FIFA world cup",
       subtitle = "Number of world cup bids compared to times hosted",
       caption = caption_cdc,
       x = "Numer of bids",
       y = "Number of times hosted")+
  theme(plot.background = element_rect(fill =  "#FAECD6"),
        panel.background = element_rect(fill =  "#FAECD6"),
        panel.grid.major = element_line(colour = "white"),
        title = element_text(colour = "#01665e"))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-41-1.png" width="100%" height="100%" />

It's now clear that Germany has the lion's share of submitted bids, while Morocco is obviously lacks a bit of luck!

What's missing from the plots above is the time where bids and hosting took place. Wouldn't it be interesting to have a single plot showing the number and dates of world cup bids? I would say YES!

![](https://media.giphy.com/media/h1oqRd3zec0MrP34Xc/giphy.gif)

Let's work towards building this exciting plot!

The data is ready for visual inspection! The idea is to look on the data in the form of a tile plot showing the year on the x axis and country on the y axis. Bids will be represented using faint colored boxes.


```r
df_bid_host1 <- tbls_lst$list_of_hosts %>%
   full_join(tbls_lst$total_bids_by_country) %>% 
  filter(!str_detect(country_name, "Cancelled")) %>% 
  arrange(bids) %>% 
  mutate(country_name = factor(country_name, levels = unique(country_name)),
         bids = factor(bids, levels = sort(unique(bids), decreasing = TRUE)))
```

```
## Joining, by = c("country_name", "country_code")
```


```r
(tile_bid_host_plt <- df_bid_host1 %>% 
  ggplot()+
  geom_tile(aes(year, country_name),
            fill = "#c7eae5", color = "white", size = 0.5) )
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-43-1.png" width="100%" height="100%" />

Next, add another layer of tiles with solid color showing the years of hosting the world cup.


```r
#select data with at least one hosting
df_bid_host2 <- df_bid_host1 %>% 
              filter(times_hosted>=1 & host_year == year)
(tile_bid_host_plt <- tile_bid_host_plt +
  geom_tile(data = df_bid_host2,
            aes(year, country_name),
            fill="#35978f", color = "black", size = 0.5))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-44-1.png" width="100%" height="100%" />

Whether you're a football fan or have an observant eye, it's not difficult to tell that there are gap years in the plot in which the world cup was cancelled. Let's highlight this part of the plot to, first, give a complete picture of the history of hosting the championship and , second, to make it clear that it's not a case of missing data.


```r
(tile_bid_host_plt <- tile_bid_host_plt +
  #add a transparent rectangle between 1942 and 1946
  geom_rect(data = tibble(xmin = 1942, xmax = 1946, ymin = -Inf, ymax = Inf),
            mapping = aes(ymin = ymin, ymax = ymax, xmin = xmin, xmax = xmax), 
            alpha = 0.05,
            fill = "black",
            color = "black",
            size = 0.1,
            inherit.aes = FALSE)+
  #overlay an explanation on top the rectangle
  annotate("text",
           angle = 90, x = 1944, y = 17.5,size = 2.5,color = "black",
           label = "World Cups of 1942 and 1946 were both cancelled because of WW2")+
  #represet years on the x axis with 4 year interval between 1930 and 2026
  scale_x_continuous(breaks = seq(1930, 2026,4)))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-45-1.png" width="100%" height="100%" />

Let's finish by adding the title and removing the y axis


```r
(tile_bid_host_plt <- tile_bid_host_plt +
  labs(title = "History of hosting FIFA world cup",
       subtitle = "Timeline of bidding (faint boxes;no outline) and hosting (dark boxes;black outline) countries of FIFA world cup",
       caption = caption_cdc)+
  theme(title = element_text(size = 9),
        axis.text.x = element_text(size = 8, angle = 45,hjust = 1),
        axis.ticks.y = element_blank(),
        axis.line.y = element_blank(),
        axis.title.y = element_blank(),
        axis.text.y = element_text(size = 7.6),
        panel.grid.major = element_line(colour = "grey80"),
        panel.grid.major.x = element_blank()))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-46-1.png" width="100%" height="100%" />

This a comprehensive, yet clear, visualization of bidding and hosting the world cup! We can simultaneously make interesting observations about the years (e.g. 1990 and 2019 received the largest number of bids!) and the history of the hosting countries (e.g. 2026 will be the first world cup to be hosted by three countries!).

![](https://media.giphy.com/media/KfJAmevRamCTGUkKUd/giphy.gif)

Let's take this plot to the next level and augment it with the results of the hosting country. Furthermore, as a cherry on top, will add respective flag of each country. To start with, we'll get the country code that would allow us to find a country's flag.


```r
#colors of the different results (First place, runner up, third place, ... etc)
res_cols <- c("#FFD700",
              "#d9d9d9",
              "#CD7F32",
              "#f6e8c3",
              "#969696",
              "#737373",
              "#525252",
              "#000000"
              )
names(res_cols) <- results_order[-length(results_order )]
```

Already tired? We're almost there!

![](https://media.giphy.com/media/J6J2RvOQLTiXI3yWKH/giphy.gif)

Let's piece everything together for one last time. We start by establishing the tiles layer and color the bids and hosts.


```r
(tile_bid_host_plt2 <- df_bid_host1 %>% 
  ggplot()+
  #tiles for bidding
  geom_tile(aes(year, country_name),
            fill = "grey85", color = "white", size = 0.5) +
  #over-plot tiles of the results
  geom_tile(data = tbls_lst$host_country_performances %>%
              filter(result != "TBD") ,
            aes(year, country_name, fill = result),
             color = "black", size = 0.5) +
  #add results colors
  scale_fill_manual(values = res_cols))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-48-1.png" width="100%" height="100%" />

We then break the x-axis by 4 years interval, add country flag, and the world cup in the background.


```r
(tile_bid_host_plt2 <- tile_bid_host_plt2+
  #define the years intervals shown on the x axis and expand left side for the flags
  scale_x_continuous(breaks = seq(1930, 2026,4),
                     expand = expansion(add = c(4,NA)))+ 
   #add country flag
  ggimage::geom_flag(data = . %>%
                       filter(!is.na(country_code)) %>%
                       distinct(country_name, country_code),
                     aes(y = country_name, image=country_code),
            x = 1925,
            size =0.03)+
   #add world cup image
  ggimage::geom_image(data = data.frame(x = 1952, y = 18),
                      aes(x,y),
                      image = wc_img,image_fun = transparent,
                      size = 1.2))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-49-1.png" width="100%" height="100%" />

Highlight and annotate the cancelled years


```r
(tile_bid_host_plt2 <- tile_bid_host_plt2 +
  #add rectangle to highlight cancelled years
  geom_rect(data = tibble(xmin = 1942, xmax = 1946, ymin = -Inf, ymax = Inf),
            mapping = aes(ymin = ymin, ymax = ymax, xmin = xmin, xmax = xmax), 
            alpha = 0.05,
            fill = "black",
            color = "black",
            size = 0.1,
            inherit.aes = FALSE)+
  #Annotate the rectangle
  annotate("text",
           angle = 90, x = 1944, y = 17.5,size = 2.5,color = "black",
           label = "World Cups of 1942 and 1946 were both cancelled because of WW2"))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-50-1.png" width="100%" height="100%" />

Finally, add the title and the control the theme


```r
(tile_bid_host_plt2 <- tile_bid_host_plt2 +
  #define title and subtitle
  labs(title = "History of hosting FIFA world cup",
       subtitle = "Timeline of bidding (faint boxes;no outline) and hosting (dark boxes;black outline) countries of FIFA world cup",
       caption = caption_cdc)+
  #control the order and size of the legend keys
  guides(fill = guide_legend(nrow = 1,
                             keywidth = 0.85, 
                             keyheight = 0.25))+
  theme(title = element_text(size = 9),
        axis.text.x = element_text(size = 8, angle = 45,hjust = 1),
        axis.ticks.y = element_blank(),
        axis.line.y = element_blank(),
        axis.title.y = element_blank(),
        axis.text.y = element_text(size = 7.6),
        panel.grid.major = element_line(colour = "grey80"),
        panel.grid.major.x = element_blank(),
        legend.title = element_blank(),
        legend.text = element_text(size = 6.9),
        legend.spacing.x = unit(0.1,"cm" ),
        legend.position = "top"
        ))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-51-1.png" width="100%" height="100%" />

WOW! We've managed to summarize the history of world cup in a single plot!

Mission accomplished!

![](https://media.giphy.com/media/WZTgoOuIs63QEV8IFn/giphy.gif)


