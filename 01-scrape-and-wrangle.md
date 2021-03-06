Don’t Mess with Texas Part 1: scraping the HTML tables
================
Martin Frigaard
2019-11-28

# Texas death row executed offenders website

Texas Department of Criminal Justice keeps records of every inmate they
execute. We’re going to scrape the data found
[here](http://www.tdcj.state.tx.us/death_row/dr_executed_offenders.html).

``` r
library(rvest)     
library(jsonlite)  
library(tidyverse) 
library(tidyquant) 
library(xopen)     
library(knitr)     
library(xml2)
```

## Scraping the data from HTML websites

Load the `xml2` package and define the url with the data (here it’s
`webpage_url`).

``` r
webpage_url <- "http://www.tdcj.state.tx.us/death_row/dr_executed_offenders.html"
webpage <- xml2::read_html(webpage_url)
```

Use the `rvest::html_table()` to find the table in the `webpage` object.
This is at position `[[1]]`.

The `dplyr::glimpse(78)` function is helpful here.

``` r
ExOffndrsRaw <- rvest::html_table(webpage)[[1]] 
# check the data.frame
ExOffndrsRaw %>% dplyr::glimpse(78)
```

    #>  Observations: 566
    #>  Variables: 10
    #>  $ Execution    <int> 566, 565, 564, 563, 562, 561, 560, 559, 558, 557, 556,…
    #>  $ Link         <chr> "Offender Information", "Offender Information", "Offen…
    #>  $ Link         <chr> "Last Statement", "Last Statement", "Last Statement", …
    #>  $ `Last Name`  <chr> "Hall", "Sparks", "Soliz", "Crutsinger", "Swearingen",…
    #>  $ `First Name` <chr> "Justen", "Robert", "Mark", "Billy", "Larry", "John", …
    #>  $ TDCJNumber   <int> 999497, 999542, 999571, 999459, 999361, 999295, 976, 9…
    #>  $ Age          <int> 38, 45, 37, 64, 48, 44, 70, 61, 43, 47, 64, 46, 51, 34…
    #>  $ Date         <chr> "11/6/2019", "9/25/2019", "9/10/2019", "9/4/2019", "8/…
    #>  $ Race         <chr> "White", "Black", "Hispanic", "White", "White", "White…
    #>  $ County       <chr> "El Paso", "Dallas", "Johnson", "Tarrant", "Montgomery…

## Fix the column names

We can see the `Link` column is repeated, which is going to be a problem
when we put these data into their own `tibble` because R doesn’t like to
repeat the column names inside a `data.frame`. We’ll address the column
names with `base::colnames()`

``` r
base::colnames(x = rvest::html_table(webpage)[[1]])
```

    #>   [1] "Execution"  "Link"       "Link"       "Last Name"  "First Name"
    #>   [6] "TDCJNumber" "Age"        "Date"       "Race"       "County"

We will use the `tibble::as_tibble()` function, but add the
`.name_repair = "unique"` argument. The `.name_repair` argument has
other options (`"check_unique"`, `"unique"`, `"universal"` and
`"minimal"`), and you can read the help files using `?as_tibble`.

In this case, `"unique"` will work just fine.

``` r
ExecutedOffenders <- rvest::html_table(webpage)[[1]] %>% 
  # repair the repeated columns
  tibble::as_tibble(.name_repair = "unique") %>% 
  # get unique names
  janitor::clean_names(case = "snake") %>% 
  # lower, snake case
  dplyr::rename(offender_info = link_2, 
                # rename these 
                last_statement = link_3)
ExecutedOffenders %>% glimpse(78)
```

    #>  Observations: 566
    #>  Variables: 10
    #>  $ execution      <int> 566, 565, 564, 563, 562, 561, 560, 559, 558, 557, 55…
    #>  $ offender_info  <chr> "Offender Information", "Offender Information", "Off…
    #>  $ last_statement <chr> "Last Statement", "Last Statement", "Last Statement"…
    #>  $ last_name      <chr> "Hall", "Sparks", "Soliz", "Crutsinger", "Swearingen…
    #>  $ first_name     <chr> "Justen", "Robert", "Mark", "Billy", "Larry", "John"…
    #>  $ tdcj_number    <int> 999497, 999542, 999571, 999459, 999361, 999295, 976,…
    #>  $ age            <int> 38, 45, 37, 64, 48, 44, 70, 61, 43, 47, 64, 46, 51, …
    #>  $ date           <chr> "11/6/2019", "9/25/2019", "9/10/2019", "9/4/2019", "…
    #>  $ race           <chr> "White", "Black", "Hispanic", "White", "White", "Whi…
    #>  $ county         <chr> "El Paso", "Dallas", "Johnson", "Tarrant", "Montgome…

## Identify the links to offender information and last statements

Download the [selector gadget app](https://selectorgadget.com/) for your
browser. You can identify the links using the [selector
gadget](https://cran.r-project.org/web/packages/rvest/vignettes/selectorgadget.html).

<img src="figs/selector_gadget.png" width="70%" />

In order to get the `nodes` from the table, we need to send `webpage`
through a few passes of `rvest` functions (`html_nodes` and `html_attr`)
with various `css` tags to get the correct URL paths. This took a few
tries and some trial and error, but eventually I was able to figure out
the the correct combinations to get the `Links` to the pages.

``` r
Links <- webpage %>% 
  # this get the links in the overflow table 
  # row
  rvest::html_nodes(".overflow tr") %>% 
  # the links
  rvest::html_nodes("a") %>% 
  # the header ref
  rvest::html_attr("href")
# check Links
Links %>% utils::head(20)
```

    #>   [1] "dr_info/halljusten.html"           "dr_info/halljustenlast.html"      
    #>   [3] "dr_info/sparksrobert.html"         "dr_info/sparksrobertlast.html"    
    #>   [5] "dr_info/solizmarkanthony.html"     "dr_info/solizmarkanthonylast.html"
    #>   [7] "dr_info/crutsingerbilly.html"      "dr_info/crutsingerbillylast.html" 
    #>   [9] "dr_info/swearingenlarry.html"      "dr_info/swearingenlarrylast.html" 
    #>  [11] "dr_info/kingjohn.html"             "dr_info/kingjohnlast.html"        
    #>  [13] "dr_info/_coble.jpg"                "dr_info/coblebillielast.html"     
    #>  [15] "dr_info/jenningsrobert.jpg"        "dr_info/jenningsrobertlast.html"  
    #>  [17] "dr_info/brazielalvin.html"         "dr_info/brazielalvinlast.html"    
    #>  [19] "dr_info/garciajoseph.html"         "dr_info/garciajosephlast.html"

Now `Links` contain:

1)  A `dr_info/` path (which makes the entire path
    `"http://www.tdcj.state.tx.us/death_row/dr_info/"`).

<!-- end list -->

``` r
xopen("http://www.tdcj.state.tx.us/death_row/dr_info/")
```

2)  Every offender has two links–one with their full name, the other
    with a `last` string attached to the back of their full name.

Something tells me if I check the `base::length()` of `Links` with the
`base::nrow()`s in `ExOffndrs`…there will be twice as many links as rows
in executed offenders.

``` r
length(Links)
```

    #>  [1] 1132

``` r
nrow(ExecutedOffenders)
```

    #>  [1] 566

Good–this is what I want. That means each row in `ExecutedOffenders` has
two links associated with their name.

### Clean up the `last` statements

The `stringr` package can help me wrangle this long vector into the
`last_pattern` logical vector, which I then use to subset the `Links`.

``` r
last_pattern <- stringr::str_detect(
                            string = Links, 
                            pattern = "last")
utils::head(Links[last_pattern])
```

    #>  [1] "dr_info/halljustenlast.html"       "dr_info/sparksrobertlast.html"    
    #>  [3] "dr_info/solizmarkanthonylast.html" "dr_info/crutsingerbillylast.html" 
    #>  [5] "dr_info/swearingenlarrylast.html"  "dr_info/kingjohnlast.html"

Check to see that `Links[last_pattern]` is same length as the number of
rows in `ExecutedOffenders`…

``` r
base::identical(x = base::length(
                        Links[last_pattern]), 
                y = base::nrow(
                                  ExecutedOffenders))
```

    #>  [1] TRUE

Great–subset the `Links` for the `last_pattern`, then give this vector a
name (`last_links`).

``` r
last_links <- Links[last_pattern]
last_links %>% utils::head(10)
```

    #>   [1] "dr_info/halljustenlast.html"       "dr_info/sparksrobertlast.html"    
    #>   [3] "dr_info/solizmarkanthonylast.html" "dr_info/crutsingerbillylast.html" 
    #>   [5] "dr_info/swearingenlarrylast.html"  "dr_info/kingjohnlast.html"        
    #>   [7] "dr_info/coblebillielast.html"      "dr_info/jenningsrobertlast.html"  
    #>   [9] "dr_info/brazielalvinlast.html"     "dr_info/garciajosephlast.html"

If I check the length of items in `last_links`, I can see there are an
identical number of rows in the data frame.

``` r
base::identical(x = base::length(last_links),
                y = base::nrow(ExecutedOffenders))
```

    #>  [1] TRUE

## Assign the `last` column to `ExecutedOffenders`

This means I can easily assign these as a new column in
`ExecutedOffenders`.

``` r
ExecutedOffenders %>% glimpse()
```

    #>  Observations: 566
    #>  Variables: 10
    #>  $ execution      <int> 566, 565, 564, 563, 562, 561, 560, 559, 558, 557, 55…
    #>  $ offender_info  <chr> "Offender Information", "Offender Information", "Off…
    #>  $ last_statement <chr> "Last Statement", "Last Statement", "Last Statement"…
    #>  $ last_name      <chr> "Hall", "Sparks", "Soliz", "Crutsinger", "Swearingen…
    #>  $ first_name     <chr> "Justen", "Robert", "Mark", "Billy", "Larry", "John"…
    #>  $ tdcj_number    <int> 999497, 999542, 999571, 999459, 999361, 999295, 976,…
    #>  $ age            <int> 38, 45, 37, 64, 48, 44, 70, 61, 43, 47, 64, 46, 51, …
    #>  $ date           <chr> "11/6/2019", "9/25/2019", "9/10/2019", "9/4/2019", "…
    #>  $ race           <chr> "White", "Black", "Hispanic", "White", "White", "Whi…
    #>  $ county         <chr> "El Paso", "Dallas", "Johnson", "Tarrant", "Montgome…

Not done yet–I need to add the beginning of the web address:

`https://www.tdcj.texas.gov/death_row/`

``` r
# test 
ExecutedOffenders %>% 
  dplyr::mutate(
    last_url = 
        paste0("https://www.tdcj.texas.gov/death_row/", 
                                  last_links)) %>% 
  dplyr::pull(last_url) %>% 
  utils::head(10)
```

    #>   [1] "https://www.tdcj.texas.gov/death_row/dr_info/halljustenlast.html"      
    #>   [2] "https://www.tdcj.texas.gov/death_row/dr_info/sparksrobertlast.html"    
    #>   [3] "https://www.tdcj.texas.gov/death_row/dr_info/solizmarkanthonylast.html"
    #>   [4] "https://www.tdcj.texas.gov/death_row/dr_info/crutsingerbillylast.html" 
    #>   [5] "https://www.tdcj.texas.gov/death_row/dr_info/swearingenlarrylast.html" 
    #>   [6] "https://www.tdcj.texas.gov/death_row/dr_info/kingjohnlast.html"        
    #>   [7] "https://www.tdcj.texas.gov/death_row/dr_info/coblebillielast.html"     
    #>   [8] "https://www.tdcj.texas.gov/death_row/dr_info/jenningsrobertlast.html"  
    #>   [9] "https://www.tdcj.texas.gov/death_row/dr_info/brazielalvinlast.html"    
    #>  [10] "https://www.tdcj.texas.gov/death_row/dr_info/garciajosephlast.html"

``` r
# assign
ExecutedOffenders <- ExecutedOffenders %>% 
  dplyr::mutate(
    last_url = 
        paste0("https://www.tdcj.texas.gov/death_row/", 
                                  last_links))
```

Now we will tidy these up into nice, clean `LastUrl` tibble.

    #>  https://www.tdcj.texas.gov/death_row/dr_info/halljustenlast.html
    #>  https://www.tdcj.texas.gov/death_row/dr_info/sparksrobertlast.html
    #>  https://www.tdcj.texas.gov/death_row/dr_info/solizmarkanthonylast.html
    #>  https://www.tdcj.texas.gov/death_row/dr_info/crutsingerbillylast.html
    #>  https://www.tdcj.texas.gov/death_row/dr_info/swearingenlarrylast.html
    #>  https://www.tdcj.texas.gov/death_row/dr_info/kingjohnlast.html

Test one of the URLs out in the browser.

``` r
xopen("https://www.tdcj.texas.gov/death_row/dr_info/swearingenlarrylast.html")
```

### Create the info pattern

Now I want the offender information links (so I omit the links with
`last` in the pattern).

``` r
info_pattern <- !stringr::str_detect(
                            string = Links, 
                            pattern = "last")
Links[info_pattern] %>% 
  utils::head() %>% 
  base::writeLines()
```

    #>  dr_info/halljusten.html
    #>  dr_info/sparksrobert.html
    #>  dr_info/solizmarkanthony.html
    #>  dr_info/crutsingerbilly.html
    #>  dr_info/swearingenlarry.html
    #>  dr_info/kingjohn.html

Check the `base::length()` to see if it’s identical to the number of
rows in `ExecutedOffenders`.

``` r
base::identical(x = base::length(Links[info_pattern]), 
                y = base::nrow(ExecutedOffenders))
```

    #>  [1] TRUE

Great\!

Check the `length()` of `info_links`

``` r
info_links <- Links[info_pattern]
base::identical(x = base::length(info_links),
                y = base::nrow(ExecutedOffenders))
```

    #>  [1] TRUE

These are also identical. Repeat the URL process from above on the
`info_url`

Now we combine this with the `https://www.tdcj.texas.gov/death_row/`
URL.

``` r
ExecutedOffenders %>% 
  dplyr::mutate(
    info_url = 
        paste0("https://www.tdcj.texas.gov/death_row/", 
                                  info_links)) %>% 
  dplyr::pull(last_url) %>% 
  utils::head(10)
```

    #>   [1] "https://www.tdcj.texas.gov/death_row/dr_info/halljustenlast.html"      
    #>   [2] "https://www.tdcj.texas.gov/death_row/dr_info/sparksrobertlast.html"    
    #>   [3] "https://www.tdcj.texas.gov/death_row/dr_info/solizmarkanthonylast.html"
    #>   [4] "https://www.tdcj.texas.gov/death_row/dr_info/crutsingerbillylast.html" 
    #>   [5] "https://www.tdcj.texas.gov/death_row/dr_info/swearingenlarrylast.html" 
    #>   [6] "https://www.tdcj.texas.gov/death_row/dr_info/kingjohnlast.html"        
    #>   [7] "https://www.tdcj.texas.gov/death_row/dr_info/coblebillielast.html"     
    #>   [8] "https://www.tdcj.texas.gov/death_row/dr_info/jenningsrobertlast.html"  
    #>   [9] "https://www.tdcj.texas.gov/death_row/dr_info/brazielalvinlast.html"    
    #>  [10] "https://www.tdcj.texas.gov/death_row/dr_info/garciajosephlast.html"

``` r
# assign
ExecutedOffenders <- ExecutedOffenders %>% 
  dplyr::mutate(
    info_url = 
        paste0("http://www.tdcj.state.tx.us/death_row/", 
                                  info_links))
```

These are complete URLs–assign this to `ExecutedOffenders` data frame.
Put the `InfoLinks` into a tidy data frame.

``` r
info_links <- Links[info_pattern]

InfoLinks <- info_links %>% 
  # turn into a tibble
  tibble::as_tibble(.name_repair = "unique") %>% 
  # tidy
  tidyr::gather(key = "key",
                value = "value") %>% 
  # rename the value
  dplyr::select(dr_info_url = value) %>% 
  # create the new url with death row root
  dplyr::mutate(
    dr_info_url = paste0("http://www.tdcj.state.tx.us/death_row/", info_links))

InfoLinks %>% dplyr::glimpse(78)
```

    #>  Observations: 566
    #>  Variables: 1
    #>  $ dr_info_url <chr> "http://www.tdcj.state.tx.us/death_row/dr_info/halljust…

Test a few of these out in the browser:

``` r
xopen("http://www.tdcj.state.tx.us/death_row/dr_info/brookscharlie.html")
```

We can see from the image below that this url works.

<img src="figs/test_url.png" width="70%" />

Now we assign these links to the `ExecutedOffenders` data frame. But
first make sure they match up.

``` r
ExecutedOffenders %>% 
  dplyr::select(last_name, 
                first_name) %>%
  utils::head(10)
```

    #>  # A tibble: 10 x 2
    #>     last_name    first_name
    #>     <chr>        <chr>     
    #>   1 Hall         Justen    
    #>   2 Sparks       Robert    
    #>   3 Soliz        Mark      
    #>   4 Crutsinger   Billy     
    #>   5 Swearingen   Larry     
    #>   6 King         John      
    #>   7 Coble        Billie    
    #>   8 Jennings     Robert    
    #>   9 Braziel, Jr. Alvin     
    #>  10 Garcia       Joseph

``` r
ExecutedOffenders %>% 
  dplyr::select(last_name, 
                first_name) %>%
  utils::tail(10)
```

    #>  # A tibble: 10 x 2
    #>     last_name   first_name
    #>     <chr>       <chr>     
    #>   1 Rumbaugh    Charles   
    #>   2 Porter      Henry     
    #>   3 Milton      Charles   
    #>   4 De La Rosa  Jesse     
    #>   5 Morin       Stephen   
    #>   6 Skillern    Doyle     
    #>   7 Barefoot    Thomas    
    #>   8 O'Bryan     Ronald    
    #>   9 Autry       James     
    #>  10 Brooks, Jr. Charlie

``` r
# Use `dplyr::bind_cols()` to attach these columns to `ExecutedOffenders` and 
# rename to`ExOffndrsComplete`
ExecutedOffenders <- ExecutedOffenders %>% 
  # add the info_url
  dplyr::bind_cols(LastUrl) %>%
  # add the
  dplyr::bind_cols(InfoLinks) %>%
  # move the names to the front
  dplyr::select(dplyr::ends_with("name"),
                # all else
                dplyr::everything())
ExecutedOffenders %>% dplyr::glimpse(78)
```

    #>  Observations: 566
    #>  Variables: 14
    #>  $ last_name      <chr> "Hall", "Sparks", "Soliz", "Crutsinger", "Swearingen…
    #>  $ first_name     <chr> "Justen", "Robert", "Mark", "Billy", "Larry", "John"…
    #>  $ execution      <int> 566, 565, 564, 563, 562, 561, 560, 559, 558, 557, 55…
    #>  $ offender_info  <chr> "Offender Information", "Offender Information", "Off…
    #>  $ last_statement <chr> "Last Statement", "Last Statement", "Last Statement"…
    #>  $ tdcj_number    <int> 999497, 999542, 999571, 999459, 999361, 999295, 976,…
    #>  $ age            <int> 38, 45, 37, 64, 48, 44, 70, 61, 43, 47, 64, 46, 51, …
    #>  $ date           <chr> "11/6/2019", "9/25/2019", "9/10/2019", "9/4/2019", "…
    #>  $ race           <chr> "White", "Black", "Hispanic", "White", "White", "Whi…
    #>  $ county         <chr> "El Paso", "Dallas", "Johnson", "Tarrant", "Montgome…
    #>  $ last_url       <chr> "https://www.tdcj.texas.gov/death_row/dr_info/hallju…
    #>  $ info_url       <chr> "http://www.tdcj.state.tx.us/death_row/dr_info/hallj…
    #>  $ name_last_url  <chr> "https://www.tdcj.texas.gov/death_row/dr_info/hallju…
    #>  $ dr_info_url    <chr> "http://www.tdcj.state.tx.us/death_row/dr_info/hallj…

## Create indicator for .html vs .jpgs

Create a binary variable to identify if this is a `.jpg` or `.html` path
and name the new data frame `ExOffndrsComplete`.

``` r
ExOffndrsComplete <- ExecutedOffenders %>% 
  dplyr::mutate(jpg_html = 
        dplyr::case_when(
          str_detect(string = info_url, pattern = ".jpg") ~ "jpg", 
          str_detect(string = info_url, pattern = ".html") ~ "html")) 
ExOffndrsComplete %>% dplyr::count(jpg_html)
```

    #>  # A tibble: 2 x 2
    #>    jpg_html     n
    #>    <chr>    <int>
    #>  1 html       186
    #>  2 jpg        380

Use `dplyr::sample_n` to check a few examples of this new variable.

``` r
ExOffndrsComplete %>% 
  dplyr::sample_n(size = 10) %>% 
  dplyr::select(info_url, 
                jpg_html) %>% 
  dplyr::count(jpg_html)
```

    #>  # A tibble: 2 x 2
    #>    jpg_html     n
    #>    <chr>    <int>
    #>  1 html         2
    #>  2 jpg          8

## Export the data with a date stamp

We now have a data frame we can export into a dated folder.

``` r
# create data folder
if (!fs::dir_exists("data")) {
  fs::dir_create("data")
}
# create processed folder 
if (!fs::dir_exists("data/processed")) {
  fs::dir_create("data/processed")
}
# create today
tahday <- as.character(lubridate::today())
tahday_path <- file.path("data/processed", tahday)
tahday_path
```

    #>  [1] "data/processed/2019-11-28"

``` r
# create new data folder
if (!fs::dir_exists(tahday_path)) {
  fs::dir_create(tahday_path)
}
# export these data
readr::write_csv(as.data.frame(ExOffndrsComplete),
                 path = paste0(tahday_path, "/", 
                               tahday,"-ExOffndrsComplete.csv"))
fs::dir_tree("data")
```

    #>  data
    #>  └── processed
    #>      ├── 2018-12-20
    #>      │   ├── 2018-12-20-ExExOffndrshtml.csv
    #>      │   ├── 2018-12-20-ExExOffndrsjpg.csv
    #>      │   └── 2018-12-20-ExOffndrsComplete.csv
    #>      ├── 2019-11-27
    #>      │   └── 2019-11-27-ExOffndrsComplete.csv
    #>      └── 2019-11-28
    #>          ├── 2019-11-28-ExExOffndrshtml.csv
    #>          ├── 2019-11-28-ExExOffndrsjpg.csv
    #>          ├── 2019-11-28-ExOffndrsComplete.csv
    #>          └── 2019-11-28-ExecOffenders.csv
