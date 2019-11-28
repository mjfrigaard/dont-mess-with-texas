Don’t Mess with Texas: data from Texas department of corrections
================

These data come from the [Texas Department of Criminal
Justice](https://www.tdcj.texas.gov/index.html) website that holds death
row information on executed
[offenders](https://www.tdcj.texas.gov/death_row/dr_executed_offenders.html).

## Executions in Texas

Capital punishment in Texas has a long history (read about it
[here](https://en.wikipedia.org/wiki/Capital_punishment_in_Texas)). At
the time of this writing (**2019-11-28**), Texas has carried out more
than 1/3 of the total executions in the United States. The project tells
the story of capitol punishment in Texas (and the US). I created this
project to raise awareness about the reality of state-sanctioned deaths,
and to try and understand more about why Texas is such an outlier with
respect to capitol punishment.

I constantly stumble across data on websites I’d like to use in a
visualization or analyze. R comes with two great packages for scraping
data from .html tables (`rvest` and `xml`). In order to download data,
sometimes each file needs to be downloaded onto your local machine. The
`purrr` package has quite a few excellent functions for iteration to
help with this.

## Texas death row executed offenders website

Texas Department of Criminal Justice keeps records of every inmate they
execute. We are going to scrape the data found
[here](http://www.tdcj.state.tx.us/death_row/dr_executed_offenders.html).
