

# History of hosting FIFA World Cup

**Overview**

An exploratory data analysis project on the history of hosting FIFA world cup.

**Outline**

1.  [Where and how can we get the data?]

    To start with, we will learn how to scrape Wikipedia directly into R, parse the data tables, and apply quality control to make them ready for the analysis.

    ![](images/aMjwwYeTrEkCyr6af-th-2721308714.jpg){width="265"}

2.  [How many world cups were hosted in each continent?]

    Will then move on to explore the number of hosted cups at the level of continents and the geographical distribution of hosting countries.

    ![](images/unnamed-chunk-12-1.png){width="490"}![](images/unnamed-chunk-16-1.png){width="490"}

3.  [What is the timeline of hosting the world cup?]

    Next, will add the time component by generating a condensed timeline of the history of hosting world cups on the level countries and continents.

    ![](images/unnamed-chunk-19-1.png){width="494"}

4.  [What is the history of bidding for world cup?]

    Finally, we will go beyond the mere hosting the championship to explore the bidding process and the performance of the hosting team over the years.

    ![](images/unnamed-chunk-20-1.png){width="497"}

    ![](images/unnamed-chunk-21-1.png){width="498"}

    ![](images/unnamed-chunk-27-1.png){width="499"}

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
#set the default ggplot theme
theme_set(cowplot::theme_cowplot())
```

and with that, we're ready to ride!

![](https://media.giphy.com/media/kgsqn9gCVAQ3YM3C2f/giphy.gif)

## Where and how can we get the data?

Generally we would like to know **who** (country, continent) *hosted* **when**. Since *hosting* is a lengthy process that starts by *bidding* and followed by FIFA evaluation. it would be interesting to incorporate *bidding* data into the analysis.

In this project we will use the data made available in this Wikipedia article about [FIFA World Cup hosts](https://en.wikipedia.org/wiki/FIFA_World_Cup_hosts)

To do that, we are going to use the [rvest](https://rvest.tidyverse.org/) package to explore and scrape this tables directly into R.


```r
# URL of the article
url <- "https://en.wikipedia.org/wiki/FIFA_World_Cup_hosts"
# Read the webpage and obtain the pieces of the article containing tables
tbls_lst <-  url %>%
  read_html %>%
  html_table()
```

The tables are in the house! However, this too much. Let's select only the tables of interest for this tutorial. This is limited to the subset of tables showing the list of countries that have submitted a bid or actually hosted the world cup. As an extra, we will also utilize the performance of host countries in our analysis.


```r
# Select tables of interest
tbls_lst <- tbls_lst[c(1,9,10)]

# Assign names to the tables
tables_names <- c("List of hosts",
                  "Total bids by country",
                  "Host country performances")
names(tbls_lst) <- tolower(tables_names) %>% str_replace_all(" ","_")
```

Let's have a quick look at the three selected tables


```r
gt::gt(head(tbls_lst$list_of_hosts))
```

```{=html}
<div id="crpctzxnyq" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#crpctzxnyq .gt_table {
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

#crpctzxnyq .gt_heading {
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

#crpctzxnyq .gt_title {
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

#crpctzxnyq .gt_subtitle {
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

#crpctzxnyq .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#crpctzxnyq .gt_col_headings {
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

#crpctzxnyq .gt_col_heading {
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

#crpctzxnyq .gt_column_spanner_outer {
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

#crpctzxnyq .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#crpctzxnyq .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#crpctzxnyq .gt_column_spanner {
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

#crpctzxnyq .gt_group_heading {
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

#crpctzxnyq .gt_empty_group_heading {
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

#crpctzxnyq .gt_from_md > :first-child {
  margin-top: 0;
}

#crpctzxnyq .gt_from_md > :last-child {
  margin-bottom: 0;
}

#crpctzxnyq .gt_row {
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

#crpctzxnyq .gt_stub {
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

#crpctzxnyq .gt_stub_row_group {
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

#crpctzxnyq .gt_row_group_first td {
  border-top-width: 2px;
}

#crpctzxnyq .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#crpctzxnyq .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#crpctzxnyq .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#crpctzxnyq .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#crpctzxnyq .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#crpctzxnyq .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#crpctzxnyq .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#crpctzxnyq .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#crpctzxnyq .gt_footnotes {
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

#crpctzxnyq .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-left: 4px;
  padding-right: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#crpctzxnyq .gt_sourcenotes {
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

#crpctzxnyq .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#crpctzxnyq .gt_left {
  text-align: left;
}

#crpctzxnyq .gt_center {
  text-align: center;
}

#crpctzxnyq .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#crpctzxnyq .gt_font_normal {
  font-weight: normal;
}

#crpctzxnyq .gt_font_bold {
  font-weight: bold;
}

#crpctzxnyq .gt_font_italic {
  font-style: italic;
}

#crpctzxnyq .gt_super {
  font-size: 65%;
}

#crpctzxnyq .gt_footnote_marks {
  font-style: italic;
  font-weight: normal;
  font-size: 75%;
  vertical-align: 0.4em;
}

#crpctzxnyq .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#crpctzxnyq .gt_indent_1 {
  text-indent: 5px;
}

#crpctzxnyq .gt_indent_2 {
  text-indent: 10px;
}

#crpctzxnyq .gt_indent_3 {
  text-indent: 15px;
}

#crpctzxnyq .gt_indent_4 {
  text-indent: 20px;
}

#crpctzxnyq .gt_indent_5 {
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
<td class="gt_row gt_left">Canceled because of World War II</td>
<td class="gt_row gt_left">Canceled because of World War II</td></tr>
    <tr><td class="gt_row gt_right">1946</td>
<td class="gt_row gt_left">Canceled because of World War II</td>
<td class="gt_row gt_left">Canceled because of World War II</td></tr>
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
<div id="lepdmahcvc" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#lepdmahcvc .gt_table {
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

#lepdmahcvc .gt_heading {
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

#lepdmahcvc .gt_title {
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

#lepdmahcvc .gt_subtitle {
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

#lepdmahcvc .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#lepdmahcvc .gt_col_headings {
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

#lepdmahcvc .gt_col_heading {
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

#lepdmahcvc .gt_column_spanner_outer {
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

#lepdmahcvc .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#lepdmahcvc .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#lepdmahcvc .gt_column_spanner {
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

#lepdmahcvc .gt_group_heading {
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

#lepdmahcvc .gt_empty_group_heading {
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

#lepdmahcvc .gt_from_md > :first-child {
  margin-top: 0;
}

#lepdmahcvc .gt_from_md > :last-child {
  margin-bottom: 0;
}

#lepdmahcvc .gt_row {
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

#lepdmahcvc .gt_stub {
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

#lepdmahcvc .gt_stub_row_group {
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

#lepdmahcvc .gt_row_group_first td {
  border-top-width: 2px;
}

#lepdmahcvc .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#lepdmahcvc .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#lepdmahcvc .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#lepdmahcvc .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#lepdmahcvc .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#lepdmahcvc .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#lepdmahcvc .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#lepdmahcvc .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#lepdmahcvc .gt_footnotes {
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

#lepdmahcvc .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-left: 4px;
  padding-right: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#lepdmahcvc .gt_sourcenotes {
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

#lepdmahcvc .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#lepdmahcvc .gt_left {
  text-align: left;
}

#lepdmahcvc .gt_center {
  text-align: center;
}

#lepdmahcvc .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#lepdmahcvc .gt_font_normal {
  font-weight: normal;
}

#lepdmahcvc .gt_font_bold {
  font-weight: bold;
}

#lepdmahcvc .gt_font_italic {
  font-style: italic;
}

#lepdmahcvc .gt_super {
  font-size: 65%;
}

#lepdmahcvc .gt_footnote_marks {
  font-style: italic;
  font-weight: normal;
  font-size: 75%;
  vertical-align: 0.4em;
}

#lepdmahcvc .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#lepdmahcvc .gt_indent_1 {
  text-indent: 5px;
}

#lepdmahcvc .gt_indent_2 {
  text-indent: 10px;
}

#lepdmahcvc .gt_indent_3 {
  text-indent: 15px;
}

#lepdmahcvc .gt_indent_4 {
  text-indent: 20px;
}

#lepdmahcvc .gt_indent_5 {
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
<div id="yhknlyzwnm" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#yhknlyzwnm .gt_table {
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

#yhknlyzwnm .gt_heading {
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

#yhknlyzwnm .gt_title {
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

#yhknlyzwnm .gt_subtitle {
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

#yhknlyzwnm .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#yhknlyzwnm .gt_col_headings {
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

#yhknlyzwnm .gt_col_heading {
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

#yhknlyzwnm .gt_column_spanner_outer {
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

#yhknlyzwnm .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#yhknlyzwnm .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#yhknlyzwnm .gt_column_spanner {
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

#yhknlyzwnm .gt_group_heading {
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

#yhknlyzwnm .gt_empty_group_heading {
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

#yhknlyzwnm .gt_from_md > :first-child {
  margin-top: 0;
}

#yhknlyzwnm .gt_from_md > :last-child {
  margin-bottom: 0;
}

#yhknlyzwnm .gt_row {
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

#yhknlyzwnm .gt_stub {
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

#yhknlyzwnm .gt_stub_row_group {
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

#yhknlyzwnm .gt_row_group_first td {
  border-top-width: 2px;
}

#yhknlyzwnm .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#yhknlyzwnm .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#yhknlyzwnm .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#yhknlyzwnm .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#yhknlyzwnm .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#yhknlyzwnm .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#yhknlyzwnm .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#yhknlyzwnm .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#yhknlyzwnm .gt_footnotes {
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

#yhknlyzwnm .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-left: 4px;
  padding-right: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#yhknlyzwnm .gt_sourcenotes {
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

#yhknlyzwnm .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#yhknlyzwnm .gt_left {
  text-align: left;
}

#yhknlyzwnm .gt_center {
  text-align: center;
}

#yhknlyzwnm .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#yhknlyzwnm .gt_font_normal {
  font-weight: normal;
}

#yhknlyzwnm .gt_font_bold {
  font-weight: bold;
}

#yhknlyzwnm .gt_font_italic {
  font-style: italic;
}

#yhknlyzwnm .gt_super {
  font-size: 65%;
}

#yhknlyzwnm .gt_footnote_marks {
  font-style: italic;
  font-weight: normal;
  font-size: 75%;
  vertical-align: 0.4em;
}

#yhknlyzwnm .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#yhknlyzwnm .gt_indent_1 {
  text-indent: 5px;
}

#yhknlyzwnm .gt_indent_2 {
  text-indent: 10px;
}

#yhknlyzwnm .gt_indent_3 {
  text-indent: 15px;
}

#yhknlyzwnm .gt_indent_4 {
  text-indent: 20px;
}

#yhknlyzwnm .gt_indent_5 {
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

Looks good, but not perfect. As expected, the scrapped tables are not optimal for analysis. Let's push the tables through a few rounds of quality control.


```r
# Clean columns' names
tbls_lst <- lapply(tbls_lst,  janitor::clean_names)
# Extract the amount of money
tbls_lst$host_country_performances  <- tbls_lst$host_country_performances %>%
  mutate(esult = str_replace(result, " \\(top 12\\)", "")) %>%
  dplyr::rename(country = "team",
                years = "year")
#let's correct the entry of Colombia since it withdrew from hosting due to economic concerns
tbls_lst$total_bids_by_country <- tbls_lst$total_bids_by_country %>% 
  mutate(times_hosted = ifelse(country == "Colombia", 0, times_hosted))
#Replace "West Germany" with "Germany"
tbls_lst$host_country_performances  <- tbls_lst$host_country_performances %>%
  mutate(team = str_replace(country, "West Germany", "Germany"))
tbls_lst$list_of_hosts <- tbls_lst$list_of_hosts %>% 
  mutate(host_nation_s = str_replace(host_nation_s, "West Germany", "Germany"))
#Order the results and set them as levels
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
tbls_lst$host_country_performances  <- tbls_lst$host_country_performances %>%
  mutate(result = ifelse(result == "Second round (top 12)", "Second round", result),
                          
    result = factor(result, levels = results_order),
         country = str_replace(country, "West Germany", "Germany")) 
```

Now that the data is analysis-ready, it is time to explore some interesting questions!

![](https://media.giphy.com/media/WmzhEJZsON3Bk7ulRY/giphy.gif)

## How many world cups were hosted in each continent?

Before embarking on our colorful journey of data visualization, let's define a caption that credits the source of the data and the analysis.


```r
caption_cdc <- glue::glue("Data source: {url}\n@Cairo Data Science Club")
theme_update(plot.caption = element_text(face = "italic"))
```

We will exclude the dates in which the championship were cancelled because of Warld War II


```r
df_host1 <- tbls_lst$list_of_hosts %>% 
  filter(!str_detect(continent, "Canceled"))
```

Let's look at a basic plot of the data


```r
(bar_host_plt <- df_host1 %>% 
  ggplot(aes(continent))+
  geom_bar())
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-8-1.png" width="100%" height="100%" />

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

<img src="01-hosting_history_files/figure-html/unnamed-chunk-9-1.png" width="100%" height="100%" />

Second, let's add some text and remove the legend since it doesn't add to the plot.


```r
(bar_host_plt <- bar_host_plt +
    labs(title = "History of hosting world cup",
       subtitle = "Number of hosted world cups and hosting countries per continents",
       caption = caption_cdc))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-10-1.png" width="100%" height="100%" />

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
            aes(label = cont_n), size = 5,nudge_y = 1)+
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

<img src="01-hosting_history_files/figure-html/unnamed-chunk-11-1.png" width="100%" height="100%" />

Finally, we'll squeeze the names of the hosting countries inside the bars of the corresponding continent


```r
(bar_host_plt <- bar_host_plt +
    geom_text(data = df_host1 %>%
              group_by(continent, host_nation_s) %>% 
              summarise(n_host = n()) %>% 
              group_by(continent) %>% 
              mutate(n_cont = n(),
                     prop = sum(n_host)/(n_cont+1),
                     cum_prop = cumsum(prop))%>%
              ungroup() %>% 
              mutate(host_nation_s = ifelse(n_host >1 , glue::glue("{host_nation_s} x {n_host}"), host_nation_s)),
            aes(y = cum_prop, label = host_nation_s),
            size = 4))
```

```
## `summarise()` has grouped output by 'continent'. You can override using the
## `.groups` argument.
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-12-1.png" width="100%" height="100%" />

Isn't that nice?!

![](https://media.giphy.com/media/UQsYRkg6DLbJ6eIrYV/giphy.gif)

Let's add some geographical-context to these results by throwing the data on the world map and see how it would look like.


```r
#get map of the world
world <- ne_countries(scale = "medium", returnclass = "sf")
#and map of separate host countries
hst_cntry <- df_host1$host_nation_s %>%
           unique() %>% 
           str_split(" \\s") %>%
           unlist()
wcp_hosts <- gisco_get_countries(country = hst_cntry,
                                 epsg = 3857# Pseudo-Mercator projection
                                 )
# Convert country name to iso2c code
wcp_hosts$iso2 <- countrycode(wcp_hosts$ISO3_CODE, "iso3c", "iso2c")
```

Plotting base map of the world using ggplot.


```r
# Base map of the world
(plot <- ggplot(world) +
  geom_sf(fill = "grey90") +
  theme_minimal() +
  theme(panel.background = element_rect(fill = "lightblue")))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-14-1.png" width="100%" height="100%" />

Additionally, let's make the map extra flashy by filling hosting countries with their maps.


```r
# get flags form this repo
flagrepo <- "https://raw.githubusercontent.com/hjnilsson/country-flags/master/png250px/"
```

Finally, we'll download and add flags to the world map


```r
# Loop and add
for (iso in wcp_hosts$iso2) {
  # Download pic and plot
  imgurl <- paste0(flagrepo, tolower(iso), ".png")
  tmpfile <- tempfile(fileext = ".png")
  download.file(imgurl, tmpfile, quiet = TRUE, mode = "wb")
  
  # Raster
  x <- wcp_hosts %>% filter(iso2 == iso)
  x_rast <- rasterpic_img(x, tmpfile, crop = TRUE, mask = TRUE)
  plot <- plot + layer_spatial(x_rast)
}

plot +
  geom_sf(data = wcp_hosts, fill = NA)+
  labs(title = "World map of FIFA world cup hosts")
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-16-1.png" width="100%" height="100%" />

I'm very happy with the end result!

![](https://media.giphy.com/media/jdBJkDYMphKaf89OTl/giphy.gif)\

Have a look at this [excellent blog](https://dieghernan.github.io/202201_maps-flags/) for more details on adding flags to maps This is where I got to know and learn this trick.

## What is the timeline of hosting the world cup?

What was missing from the previous representation of the data is the time component. In this section we'll explore a visualization method that would allow us to add this crucial aspect.

Let's start by preparing the hosting data for visualization by filling in the gap years where the world cup stopped due to WWII


```r
#add missing years in which world cup was cancelled
df_tm <- df_host1 %>% 
  complete(year = full_seq(year, 4)) %>% 
  mutate(continent = ifelse(is.na(continent), "Cancelled", continent),
         host_nation_s = ifelse(is.na(host_nation_s), "Cancelled", host_nation_s))
#make a 6x5 grid from the hosting data and add the coordinate of each cell in the grid
df_tm <- df_tm[1:30,] %>% 
  mutate(y = rep(6:1, each = 5),
         x = rep(1:5, 6),
         host_nation_s = case_when(
           str_detect(host_nation_s, "Canada") ~ "Canada\nMexico\nUnited States", #add a new line between cohosts
           str_detect(host_nation_s, "Japan") ~ "Japan\nSouth Korea", #add new line between cohosts
           TRUE ~ host_nation_s),
         continent = factor(continent, levels = unique(continent))) %>% 
  filter(!is.na(continent))
```

Let's use chronologically ordered tiles (AKA waffle plot) to look at the timeline of hosting the world cup.


```r
df_tm %>% 
  ggplot(aes(x, y, fill = continent ))+
  geom_tile(color = "black", size = 1)+
  geom_text(aes(label = host_nation_s), size = 4.5)+
  scale_color_manual(values = conti_cols)+
  scale_fill_manual(values = conti_cols)
```

```
## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
## ℹ Please use `linewidth` instead.
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-18-1.png" width="100%" height="100%" />

That's a good start! Let's add the year information and further beautify the plot.


```r
df_tm %>% 
  ggplot(aes(x, y, fill = continent ))+
  geom_tile(color = "black", size = 1)+
  geom_text(aes(label = host_nation_s), size = 3.1)+
  #add the time interval of each row
  geom_text(data = . %>% 
              group_by(y) %>% 
              mutate(range = glue::glue("({min(year)}-{max(year)})")) %>% 
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

<img src="01-hosting_history_files/figure-html/unnamed-chunk-19-1.png" width="100%" height="100%" />

This is a condensed and clear representation of our data, which are two desirable features of data visualization.

![](https://media.giphy.com/media/TgL7lPFfyAFITuszOA/giphy.gif)

## What is the history of bidding for world cup?

Now let us shift our focus to yet another interesting question. In this section we will explore the bidding history to host the world cup.

Let's start by making a bar-plot that show the number of successful bids for each country of all submitted bids.

First, sort the countries based on the number of bids, then the times hosted.


```r
(bar_bid_df1 <- tbls_lst$total_bids_by_country %>% 
  arrange(bids,times_hosted) %>% 
  mutate(country = factor(country, levels = (country))) )
```

```
## # A tibble: 35 × 4
##    country   bids years   times_hosted
##    <fct>    <int> <chr>          <dbl>
##  1 Austria      1 1990               0
##  2 Belgium      1 2018[f]            0
##  3 Egypt        1 2010               0
##  4 Greece       1 1990               0
##  5 Hungary      1 1930               0
##  6 Iran         1 1990               0
##  7 Libya        1 2010[h]            0
##  8 Nigeria      1 2010               0
##  9 Peru         1 1970               0
## 10 Portugal     1 2018[d]            0
## # … with 25 more rows
```

Next, add a layer of bars showing the times of bids using a transparent color


```r
(bar_bid_plt <- bar_bid_df1 %>% 
  ggplot()+
  geom_col(aes(country,bids), fill = "#35978f", alpha = 0.3))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-21-1.png" width="100%" height="100%" />

Then, add yet another layer of bars showing the times hosted using solid version of the same color


```r
(bar_bid_plt <-bar_bid_plt +
  geom_col(aes(country,times_hosted), fill = "#35978f", alpha = 1)+
  coord_flip())
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-22-1.png" width="100%" height="100%" />

Finally, add text and customize the theme


```r
(bar_bid_plt +
  labs(title = "History of hosting FIFA world cup",
     subtitle = "Number of world cup bids compared to times hosted",
     caption = caption_cdc,
       y = "Numer of bids")+
  theme(axis.title.y = element_blank(),
      axis.text.y = element_text(size = 7.6),
      plot.background = element_rect(fill =  "#FAECD6"),
        panel.background = element_rect(fill =  "#FAECD6"),
        title = element_text(colour = "#01665e", size = 9)))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-23-1.png" width="100%" height="100%" />

Nice! The plot shows that most of the time, it takes more than one bid to host the world cup.

![](https://media.giphy.com/media/mxCK3EcADG1F7krLP6/giphy.gif)

Let's enforce the relationship between bids and times hosted using a point plot.

First, let's plot the number of bids on the x-axis and the time hosted on the y-axis


```r
(point_bid_plt <- tbls_lst$total_bids_by_country %>% 
  ggplot(aes(bids, times_hosted))+
  geom_point(color = "#35978f"))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-24-1.png" width="100%" height="100%" />

Next, let's add the name of the country to the repreresentitive point


```r
(point_bid_plt <- point_bid_plt+
  geom_text(aes(label = country),
            size = 4))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-25-1.png" width="100%" height="100%" />

Oh no! Since many countries share the same bids and hosting statitics, we end up with a dramatic case of text over-plotting.

![](https://i.gifer.com/3esj.gif)

To overcome this, we'll replace `geom_text()` with `geom_text_repel()` from the package `ggrepel`. Let's first look at the effect of this function and then explain what it does.


```r
#remove the last layer added of geom_text() before using geom_text_repel()
point_bid_plt$layers[[2]] <- NULL
(point_bid_plt <- point_bid_plt+
  ggrepel::geom_text_repel(aes(label = country),
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

<img src="01-hosting_history_files/figure-html/unnamed-chunk-26-1.png" width="100%" height="100%" />

As the name says, `geom_text_repel()` makes the text repel away from each other to avoid over-plotting. The text also repel away from the edges of the plot. To avoid the undesired effect of later, we expanded the plotting are using the function `expansion()` in x and y direction

Finally, add the title and beautify the plot by coloring the background


```r
(point_bid_plt <- point_bid_plt+
  labs(title = "History of hosting FIFA world cup",
       subtitle = "Number of world cup bids compared to times hosted",
       caption = caption_cdc,
       x = "Numer of bids",
       y = "Number of times hosted")+
  theme(plot.background = element_rect(fill =  "#FAECD6"),
        panel.background = element_rect(fill =  "#FAECD6"),
        panel.grid.major = element_line(colour = "white"),
        title = element_text(colour = "#01665e")))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-27-1.png" width="100%" height="100%" />

It's now clear that Germany has the lion's share of submitted bids, while Morocco is obviously lacks a bit of luck!

What's missing from the plots above is the time where bids and hosting took place. Wouldn't it be interesting to have a single plot showing the number and dates of world cup bids? I would say YES!

![](https://media.giphy.com/media/h1oqRd3zec0MrP34Xc/giphy.gif)

Let's work towards building this exciting plot! Firs, what we need to do is to split years of bids into separate entries.


```r
#separate concatenated years into separate rows
df_bids <- tbls_lst$total_bids_by_country %>% 
  mutate(years = str_extract_all(years, "[0-9]+")) %>% 
  unnest(years) %>% 
  mutate(years = as.numeric(years))
#another way to achieve the same results using seprata_rows() and enginered regular expression
df_bids_2 <- tbls_lst$total_bids_by_country %>% 
  separate_rows(years, sep = "[^[:digit:].]+") %>% 
  filter(!nchar( years)== 0) %>% 
  mutate(years = as.numeric(years)) 
#Are both tables equal?
dplyr::all_equal(df_bids, df_bids_2)
```

```
## [1] TRUE
```

```r
rm(df_bids_2)
```

Will then do a similar thing by spliting cohosts of the same world cup (e.g. "Japan South Korea") into separate rows entries ("Japan", "South Korea").


```r
#separate cohosting countries into separate entries
df_host_2 <- df_host1 %>% 
  mutate(host_nation_s = str_split(host_nation_s, "\\s{2}")) %>%
  unnest(host_nation_s) %>% 
  dplyr::rename(country="host_nation_s",
                host_year = "year") %>%
  full_join(df_bids, by = "country") %>% 
  mutate(host_year = ifelse(is.na(continent), years, host_year)) %>% 
  arrange(bids) %>% 
  mutate(country = factor(country, levels = unique(country)),
         bids = factor(bids, levels = sort(unique(bids), decreasing = TRUE)))
```

The data is ready for visual inspection! The idea is to look on the data in the form of a tile plot showing the year on the x axis and country on the y axis. Bids will be represented using faint colored boxes.


```r
(tile_bid_host_plt <- df_host_2 %>% 
  ggplot()+
  geom_tile(aes(years, country),
            fill = "#c7eae5", color = "white", size = 0.5) )
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-30-1.png" width="100%" height="100%" />

Next, add another layer of tiles with solid color showing the years of hosting the world cup.


```r
#select data with at least one time hosting
df_host_3 <- df_host_2 %>% 
              filter(times_hosted>=1 & host_year == years)
(tile_bid_host_plt <- tile_bid_host_plt +
  geom_tile(data = df_host_3,
            aes(years, country),
            fill="#35978f", color = "black", size = 0.5))
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-31-1.png" width="100%" height="100%" />

Whether you're a football fan or have an observant eye, it's not difficult to tell that there are gap years in the plot in which the world cup was cancelled. Let's highlight this part of the plot to, first, give a complete picture of the history of hosting the championship and , second, to make itclear that it's not a case of missing data.


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

<img src="01-hosting_history_files/figure-html/unnamed-chunk-32-1.png" width="100%" height="100%" />

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

<img src="01-hosting_history_files/figure-html/unnamed-chunk-33-1.png" width="100%" height="100%" />

This a comprehensive, yet clear, visualization of bidding and hosting the world cup! We can simultaneously make interesting observations about the years (e.g. 1990 and 2019 received the largest number of bids!) and the history of the hosting countries (e.g. 2026 will be the first world cup to be hosted by three countries!).

![](https://media.giphy.com/media/KfJAmevRamCTGUkKUd/giphy.gif)

Let's take this plot to the next level and augment it with the results of the hosting country. Furthermore, as a cherry on top, will add respective flag of each country. To start with, we'll get the country code that would allow us to find a country's flag.


```r
#get iso2 code of each country (flags for Yugoslavia and England are missing)
df_host_2$iso2 <- countrycode(df_host_2$country, "country.name", "iso2c")
```

```
## Warning in countrycode_convert(sourcevar = sourcevar, origin = origin, destination = dest, : Some values were not matched unambiguously: Yugoslavia, England
```

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

Let's piece everything together for one last time.


```r
df_host_2 %>% 
  ggplot()+
  #tiles for bidding
  geom_tile(aes(years, country),
            fill = "grey85", color = "white", size = 0.5) +
  #over-plot tiles of the results
  geom_tile(data = tbls_lst$host_country_performances %>%
              filter(result != "TBD") ,
            aes(years, country, fill = result),
             color = "black", size = 0.5) +
  #add country flag
  ggimage::geom_flag(data = df_host_2 %>% distinct(country,iso2),
            aes(y = country, image=iso2),
            x = 1925,
            size =0.03)+
  #add results colors
  scale_fill_manual(values = res_cols)+
  #define the years intervals shown on the x axis and expand left side for the flags
  scale_x_continuous(breaks = seq(1930, 2026,4),
                     expand = expansion(add = c(4,NA)))+
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
           label = "World Cups of 1942 and 1946 were both cancelled because of WW2")+
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
        )
```

```
## Warning: Removed 2 rows containing missing values (`geom_image()`).
```

<img src="01-hosting_history_files/figure-html/unnamed-chunk-35-1.png" width="100%" height="100%" />

WOW! We've managed to summarize the history of world cup in a single plot!

Mission accomplished!

![](https://media.giphy.com/media/WZTgoOuIs63QEV8IFn/giphy.gif)


