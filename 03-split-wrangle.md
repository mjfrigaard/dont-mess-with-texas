Don’t Mess with Texas Part 3: Split and wrangle the data
================
Martin Frigaard
2019-11-28

# 

## The data

These data are imported from the .Rmd we used to scrape the website.
These data are in the folder below.

``` r
DirProcessed <- fs::dir_tree("data/processed") %>% 
  as_tibble() %>% 
  dplyr::arrange(desc(value))
```

    #>  data/processed
    #>  ├── 2018-12-20
    #>  │   ├── 2018-12-20-ExExOffndrshtml.csv
    #>  │   ├── 2018-12-20-ExExOffndrsjpg.csv
    #>  │   └── 2018-12-20-ExOffndrsComplete.csv
    #>  ├── 2019-11-27
    #>  │   └── 2019-11-27-ExOffndrsComplete.csv
    #>  └── 2019-11-28
    #>      ├── 2019-11-28-ExExOffndrshtml.csv
    #>      ├── 2019-11-28-ExExOffndrsjpg.csv
    #>      ├── 2019-11-28-ExOffndrsComplete.csv
    #>      └── 2019-11-28-ExecOffenders.csv

This will import the most recent data.

``` r
ExecOffenders <- readr::read_csv(DirProcessed[[1]][1])
```

    #>  Parsed with column specification:
    #>  cols(
    #>    last_name = col_character(),
    #>    first_name = col_character(),
    #>    execution = col_double(),
    #>    offender_info = col_character(),
    #>    last_statement = col_character(),
    #>    tdcj_number = col_double(),
    #>    age = col_double(),
    #>    date = col_character(),
    #>    race = col_character(),
    #>    county = col_character(),
    #>    last_url = col_character(),
    #>    info_url = col_character(),
    #>    name_last_url = col_character(),
    #>    dr_info_url = col_character(),
    #>    jpg_html = col_character()
    #>  )

## Use `purrr` and `dplyr` to split and export .csv files

This next use of `purrr` and iteration will cover how to:

1.  Split the `ExecOffenders` data frame into `ExExOffndrshtml` and
    `ExExOffndrsjpg`

2.  Save each of these data frames as .csv files

We should have two datasets with the following counts.

``` r
ExecOffenders %>% 
  dplyr::count(jpg_html, sort = TRUE)
```

    #>  # A tibble: 1 x 2
    #>    jpg_html     n
    #>    <chr>    <int>
    #>  1 jpg        380

These are new experimental functions from `dplyr`, and a big shout out
to Luis Verde Arregoitia for [his
post](https://luisdva.github.io/rstats/export-iteratively/) on a similar
topic.

The `dplyr::group_split()` *“returns a list of tibbles. Each tibble
contains the rows of .tbl for the associated group and all the columns,
including the grouping variables”*, and I combine it with
`purrr::walk()` and `readr::write_csv()` to export each file.

``` r
ExecOffenders %>% 
  dplyr::group_by(jpg_html) %>% 
  dplyr::group_split() %>% 
  purrr::walk(~.x %>% # we now carry this little .x everywhere we want it 
                      # to go.
                write_csv(path = paste0("data/", 
                                        # processed data folder
                                        "processed/",
                            # datestamp
                            base::noquote(lubridate::today()),
                            # folder
                            "/",
                            # datestamp
                            base::noquote(lubridate::today()),
                            # name of file
                            "-ExExOffndrs",
                            # split by this variable
                            base::unique(.x$jpg_html), 
                            # file extension
                            ".csv")))

fs::dir_ls("data/processed/2019-11-28")
```

    #>  data/processed/2019-11-28/2019-11-28-ExExOffndrshtml.csv
    #>  data/processed/2019-11-28/2019-11-28-ExExOffndrsjpg.csv
    #>  data/processed/2019-11-28/2019-11-28-ExOffndrsComplete.csv
    #>  data/processed/2019-11-28/2019-11-28-ExecOffenders.csv

### End
