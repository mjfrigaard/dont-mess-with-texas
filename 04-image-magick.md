Manipulating images with `magick`
================
Martin Frigaard
2019-11-28

# Texas death row executed offenders website

This continues with the [Texas Department of Criminal Justice
data](http://www.tdcj.state.tx.us/death_row/dr_executed_offenders.html),
which keeps records of every inmate executed.

We will load previous .csv file of all executions.

## Import the data

``` r
DirProcessed <- fs::dir_tree("data/processed") %>% 
  tibble::enframe(name = NULL) %>% 
  dplyr::arrange(desc(value))
```

    data/processed
    ├── 2018-12-20
    │   ├── 2018-12-20-ExExOffndrshtml.csv
    │   ├── 2018-12-20-ExExOffndrsjpg.csv
    │   └── 2018-12-20-ExOffndrsComplete.csv
    ├── 2019-11-27
    │   └── 2019-11-27-ExOffndrsComplete.csv
    └── 2019-11-28
        ├── 2019-11-28-ExExOffndrshtml.csv
        ├── 2019-11-28-ExExOffndrsjpg.csv
        ├── 2019-11-28-ExOffndrsComplete.csv
        └── 2019-11-28-ExecOffenders.csv

This will import the most recent data.

``` r
ExecOffenders <- readr::read_csv(DirProcessed[[1]][1])
ExecOffenders %>% skimr::skim()
```

|                                                  |            |
| :----------------------------------------------- | :--------- |
| Name                                             | Piped data |
| Number of rows                                   | 380        |
| Number of columns                                | 15         |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   |            |
| Column type frequency:                           |            |
| character                                        | 12         |
| numeric                                          | 3          |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ |            |
| Group variables                                  | None       |

Data summary

**Variable type: character**

| skim\_variable  | n\_missing | complete\_rate | min | max | empty | n\_unique | whitespace |
| :-------------- | ---------: | -------------: | --: | --: | ----: | --------: | ---------: |
| last\_name      |          0 |              1 |   3 |  15 |     0 |       325 |          0 |
| first\_name     |          0 |              1 |   3 |   9 |     0 |       202 |          0 |
| offender\_info  |          0 |              1 |  20 |  20 |     0 |         1 |          0 |
| last\_statement |          0 |              1 |  14 |  14 |     0 |         1 |          0 |
| date            |          0 |              1 |   9 |  10 |     0 |       377 |          0 |
| race            |          0 |              1 |   5 |   8 |     0 |         4 |          0 |
| county          |          0 |              1 |   3 |  12 |     0 |        77 |          0 |
| last\_url       |          0 |              1 |  61 |  73 |     0 |       297 |          0 |
| info\_url       |          0 |              1 |  56 |  69 |     0 |       380 |          0 |
| name\_last\_url |          0 |              1 |  61 |  73 |     0 |       297 |          0 |
| dr\_info\_url   |          0 |              1 |  56 |  69 |     0 |       380 |          0 |
| jpg\_html       |          0 |              1 |   3 |   3 |     0 |         1 |          0 |

**Variable type: numeric**

| skim\_variable | n\_missing | complete\_rate |      mean |        sd |  p0 |    p25 |   p50 |      p75 |   p100 | hist  |
| :------------- | ---------: | -------------: | --------: | --------: | --: | -----: | ----: | -------: | -----: | :---- |
| execution      |          0 |              1 |    218.17 |    132.09 |   3 | 110.75 | 208.5 |    309.5 |    560 | ▇▇▇▃▂ |
| tdcj\_number   |          0 |              1 | 350210.07 | 476794.93 | 511 | 752.75 | 916.5 | 999071.2 | 999234 | ▇▁▁▁▅ |
| age            |          0 |              1 |     40.08 |      8.87 |  24 |  33.00 |  39.0 |     45.0 |     70 | ▅▇▃▂▁ |

## The `magik` package

I will be using the
[magik](https://cran.r-project.org/web/packages/magick/vignettes/intro.html)
package for processing and manipulating these images. I advise checking
out the entire vignette.

## Create a test image

Convert this to a `magick` image using the `image_read()` function. The
code below selects a jpg at random, and If I print this within the
Rmarkdown file, I see the output in the viewer pane.

``` r
library(magick)
fs::dir_ls("jpgs") %>% 
  tibble::enframe(name = NULL) %>% 
  dplyr::filter(stringr::str_detect(string = value,
                                    pattern = "shields")) %>% 
  as.character() %>% 
  magick::image_read() -> test_image
```

    Error in magick_image_readpath(enc2native(path), density, depth, strip): Magick: UnableToOpenBlob `character(0)': No such file or directory @ error/blob.c/OpenBlob/2761

``` r
test_image
```

    Error in eval(expr, envir, enclos): object 'test_image' not found

I store this as an object in R named `test_image`.

## Read, write, join, or combine (`image_read`)

I create `test_magick_img` from `magick::image_read()`, and then go on
making the transformations as necessary.

``` r
# fs::dir_ls("figs")
test_magick_img <- magick::image_read(paste0(
              "figs/","shieldsrobert.jpg"))
test_magick_img
```

![](04-image-magick_files/figure-gfm/test_magick_img-1.png)<!-- -->

*TIP: come up with a naming convention for each step so you can use
RStudio’s viewer pane to see the manipulations.*

## Basic transformations

These functions are for basic image movement/manipulations you would do
with any basic photo editing app.

### Crop with `magick::image_crop()`

Now I want to remove the text and focus on the mugshot. This might need
to be adjusted slightly for each new `test_magick_img`.

``` r
# crop this image
test_magick_crop1 <- magick::image_crop(image = test_magick_img, 
                                      geometry = "750x1000+10")
test_magick_crop1
```

![](04-image-magick_files/figure-gfm/test_magick_crop-1.png)<!-- -->

This should have trimmed the extra space off the bottom of the image.

### Rotate with `magick::image_rotate()`

I want to rotate this image by 90 degrees.

``` r
# rotate this image
test_magick_rotate90 <- magick::image_rotate(test_magick_crop1, 
                                          degrees = 90)
test_magick_rotate90
```

![](04-image-magick_files/figure-gfm/mwe_magick_rotate-1.png)<!-- -->

This is what it looks like in RStudio.

![](figs/magik-image-rotate90.png)<!-- -->

Now I want to remove the rest of the text and focus on the mugshot. This
might need to be adjusted slightly for each new `test_image`.

``` r
# crop this image
test_magick_crop2 <- magick::image_crop(image = test_magick_rotate90, 
                                      geometry = "850x590+01")
test_magick_crop2
```

![](04-image-magick_files/figure-gfm/test_magick_crop2-1.png)<!-- -->

Now I will rotate this image back to center and flip it using
`magick::image_flip()`

``` r
# rotate this image
test_magick_rotate270 <- magick::image_rotate(test_magick_crop2, 
                                          degrees = 270)
# rotate this image
test_magick_flip <- magick::image_flip(test_magick_rotate270)
test_magick_flip
```

![](04-image-magick_files/figure-gfm/test_magick_rotate270-test_magick_flip-1.png)<!-- -->

![](figs/magik-image-flip.png)<!-- -->

I’ll crop the rest of the text out of the image, and trim the whitespace
for the plot.

``` r
# crop this image
test_magick_crop3 <- magick::image_crop(image = test_magick_flip, 
                                      geometry = "750x200+10")
# test_magick_crop3
# flip this image again
test_magick_flip2 <- magick::image_flip(test_magick_crop3)
# test_magick_flip2
# rotate to remove the dot
test_magick_rotate270v2 <-  magick::image_rotate(test_magick_flip2, 
                                                 degrees = 270)
# test_magick_rotate270v2
# crop the dot out
test_magick_crop4 <- magick::image_crop(image = test_magick_rotate270v2, 
                                      geometry = "550x410+10")
# test_magick_crop4
# rotate back to center
test_magick_rotate90v02 <-  magick::image_rotate(test_magick_crop4, 
                                                 degrees = 90)
# test_magick_rotate90v02

# remove white background
# Here we will trim the image up a bit with the `fuzz` argument
test_magick_clean <- magick::image_trim(image = test_magick_rotate90v02, 
                                       fuzz = 1)
test_magick_clean
```

![](04-image-magick_files/figure-gfm/test_magick_crop2-test_magick_flip2-1.png)<!-- -->

Now that I have all the trimming on and cropping done, I will add some
effects for the `ggplot2` image. I want the image to be a bit more
subdued, so I will use `magick::image_modulate()` and
`magick::image_flatten()` to create these effects.

``` r
test_image_modulate <- magick::image_modulate(test_magick_clean, 
                       brightness = 100, 
                       saturation = 25, 
                       hue = 20)
# test_image_modulate
test_magick_final <- magick::image_flatten(test_image_modulate, 
                                           operator = "Threshold")
test_magick_final
```

![](04-image-magick_files/figure-gfm/test_magick_final-1.png)<!-- -->

## Data for plot

I want to graph the number of executions over time (year) by race. I can
do this by getting a grouped data from using `dplyr`’s functions.

``` r
ExOffndByRaceYear <- ExOffndrsComplete %>% 
  dplyr::filter(race != "Other") %>% 
    dplyr::mutate(
        year = lubridate::year(date),
        yday = lubridate::yday(date),
        month = lubridate::month(date, 
                      label = TRUE)) %>% 
    dplyr::group_by(race, year) %>% 
      dplyr::summarise(
            ex_x_race_year = sum(n())) %>% 
    dplyr::arrange(desc(ex_x_race_year)) 
```

    Error in eval(lhs, parent, parent): object 'ExOffndrsComplete' not found

``` r
ExOffndByRaceYear %>% glimpse(78)
```

    Error in eval(lhs, parent, parent): object 'ExOffndByRaceYear' not found

## Plot executions over time

I create `base_ggplot2` as the basic plot I want as a layer for the
image to appear on top of.

``` r
base_ggplot2 <- ExOffndByRaceYear %>% 
  ggplot2::ggplot(aes(y = ex_x_race_year, 
                      x = year, 
                      color = race)) 
```

    Error in eval(lhs, parent, parent): object 'ExOffndByRaceYear' not found

``` r
base_ggplot2 +
ggplot2::geom_line(aes(linetype = race)) +
        ggplot2::theme(legend.position = "bottom", 
                     legend.direction = "horizontal", 
                     legend.title = element_blank()) + 
      scale_x_continuous(breaks = seq(1982, 2018, 5)) +
          ggplot2::labs(
              title = "Texas Justice",
          subtitle = "Number of executions (1982-2018) in Texas",
caption = "source: http://www.tdcj.state.tx.us/death_row/index.html",
x = NULL,
y = "Executions")
```

    Error in eval(expr, envir, enclos): object 'base_ggplot2' not found

### Example 1: overplot using `grid` package

The first example I’ll plot will use image as the ‘canvas’. This
requires exporting the image as a .jpeg, then reloading it and using the
[`ggpubr`](http://www.sthda.com/english/articles/24-ggpubr-publication-ready-plots/)
package.

``` r
library(jpeg)
# 1) export the `mwe_magick_trim` file,
magick::image_write(test_magick_final, 
                    path =
                    paste0("figs/",
                           base::noquote(lubridate::today()),
                           "-test_magick_final",
                    format = ".jpg"))
# 2) then read it back in as an `jpeg::readJPEG()`.
# fs::dir_ls("image")
imgJPEG <- jpeg::readJPEG("figs/2018-12-20-test_magick_final.jpg")
```

Now I can add the `imgJPEG` after the base layer (but before I map the
`geom_line()` and `geom_theme()`).

``` r
library(ggpubr)
base_ggplot2 + 
  # this is the image for the background
        ggpubr::background_image(imgJPEG) +
  
  # here is the line graph
  ggplot2::geom_line(aes(linetype = race)) +
        ggplot2::theme(legend.position = "bottom", 
                     legend.direction = "horizontal", 
                     legend.title = element_blank()) + 
      scale_x_continuous(breaks = seq(1982, 2018, 5)) +
  # add some labels
          ggplot2::labs(
              title = "A Face in the Crowd",
          subtitle = "The total number of executions (1982-2018) in Texas",
caption = "source: http://www.tdcj.state.tx.us/death_row/index.html",
x = NULL,
y = "Executions")
```

    Error in eval(expr, envir, enclos): object 'base_ggplot2' not found

### Example 2: add this image as an annoation using `grid` package

I think a better option is to zero in on when this offender was executed
using an annotation. If you call, the original image showed a bit of
information about this offender.

``` r
test_magick_img
```

A quick Google search tells me when he was executed:

[Status: Executed by lethal injection in Texas on
January 22, 2003](http://murderpedia.org/male.L/l1/lookingbill-robert.htm)

## Edit image for annotation

I want the mugshot to show up on the graph around that date. This will
take some additional resizing, and rotating,

### Annotate images with `magick::image_annotate()`

I added an annotation (`magick::image_annotate()`) to the image and made
it transparent with `magick::image_transparent()`.

``` r
test_magick_resize <- magick::image_scale(test_magick_final, "x500") # height: 300px
# test_magick_resize
# rotate to remove the text ----
test_magick_rotate270v3 <-  magick::image_rotate(test_magick_resize, 
                                                 degrees = 270)
# test_magick_rotate270v3
# crop side view out of picture ----
test_magick_crop4 <- magick::image_crop(image = test_magick_rotate270v3, 
                                      geometry = "750x360+10")
# test_magick_crop4
# rotate again to clean up line at top of image
test_magick_rotate270v4 <-  magick::image_rotate(test_magick_crop4, 
                                                 degrees = 270)
# test_magick_rotate270v4
# crop out line
test_magick_crop5 <- magick::image_crop(image = test_magick_rotate270v4, 
                                      geometry = "750x400+10")
# test_magick_crop5
# rotate back ---
test_magick_rotate90v03 <-  magick::image_rotate(test_magick_crop5, 
                                                 degrees = 180)
# test_magick_rotate90v03
test_magick_annotate <- magick::image_annotate(image = test_magick_rotate90v03, 
               text = "EXECUTED", 
               size = 50, 
               degrees = 60,
               color = "red",
               location = "+100+90")
# test_magick_annotate
test_magick_transparent <- magick::image_transparent(test_magick_annotate, color = "white")
test_magick_transparent
```

![](04-image-magick_files/figure-gfm/test_magick_annotate-1.png)<!-- -->

This is what I see in RStudio.

![](figs/magick-annotate.png)<!-- -->

Now I create another plot with the grouped data frame.

``` r
# create plot
test_magick_raster_plot <- base_ggplot2 +
      ggplot2::geom_line(aes(linetype = race), size = 0.8) +
        ggplot2::theme(legend.position = "bottom", 
                     legend.direction = "horizontal", 
                     legend.title = element_blank()) + 
  
      scale_x_continuous(breaks = seq(1982, 2018, 5)) +
  
      ggplot2::scale_color_manual(
              labels = c("Black",
                        "Hispanic",
                        "White"),
              
              values = c("#C1CDCD", 
                         "#0A0A0A", 
                         "#8B8B83")) +
  
      ggplot2::theme(
        legend.position = "top",
              plot.background = 
                      ggplot2::element_rect(fill = NA, 
                                            color = NA),
              panel.background =
                      ggplot2::element_rect(fill = NA), 
              strip.background =
                      ggplot2::element_rect(fill = "black", 
                                          color = NA, 
                                          size = 1),
              strip.text =
                      ggplot2::element_text(colour = "white")) +
          ggplot2::labs(
          title = "A Face in the Crowd",
          subtitle = "The total number of executions (1982-2018) in Texas",
caption = "source: http://www.tdcj.state.tx.us/death_row/index.html",
x = NULL,
y = "Executions")
```

    Error in eval(expr, envir, enclos): object 'base_ggplot2' not found

``` r
test_magick_raster_plot
```

    Error in eval(expr, envir, enclos): object 'test_magick_raster_plot' not found

And convert the image 1) with `magick::image_fill()` and then to a
raster 2) with `grDevices::as.raster()`.

``` r
# convert to image fill
test_magick_fill <- magick::image_fill(test_magick_transparent, 'none')
# convert to raster
test_magick_raster <- grDevices::as.raster(test_magick_fill)
test_magick_raster_plot + 
  annotation_raster(test_magick_raster, 
                    xmin = 2001, 
                    xmax = 2006, 
                    ymin = 5, 
                    ymax = 10)
```

    Error in eval(expr, envir, enclos): object 'test_magick_raster_plot' not found

Great\! I will add more visualizations in the next post when look the
.html data.

-----

REFERENCES:

1.  Check out [this great
    post](http://bradleyboehmke.github.io/2015/12/scraping-html-text.html)
    from Bradley Boehmke to learn more about scraping html data.

2.  Check out [this video](https://www.youtube.com/watch?v=tHszX31_r4s)
    with Hadley Wickham and Andrew Ba Tran.
