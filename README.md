
This branch of the ggseg project uses `ggregions` to create an alternate
API for newcomers to brain regions.

First, we make this data simpler (collapse the sub-regions) so that it
can work with the ggregions assumptions.

``` r
library(ggregions)
library(ggplot2)
library(dplyr)
#> 
#> Attaching package: 'dplyr'
#> The following objects are masked from 'package:stats':
#> 
#>     filter, lag
#> The following objects are masked from 'package:base':
#> 
#>     intersect, setdiff, setequal, union

coronal_ref_data <- ggseg::aseg$data |> 
  filter(side == "coronal") |>
  group_by(region) |> 
  summarise(geometry = sf::st_combine(geometry)) |> 
  select(region, everything())
  
head(coronal_ref_data)
#> Simple feature collection with 6 features and 1 field
#> Geometry type: MULTIPOLYGON
#> Dimension:     XY
#> Bounding box:  xmin: 0.30664 ymin: 0.37402 xmax: 2.21213 ymax: 2.00457
#> CRS:           NA
#> # A tibble: 6 × 2
#>   region                                                                geometry
#>   <chr>                                                           <MULTIPOLYGON>
#> 1 amygdala          (((1.92373 0.93533, 1.93033 0.93137, 1.99653 0.89127, 2.003…
#> 2 caudate           (((0.72747 1.86448, 0.73377 1.86329, 0.75194 1.85575, 0.770…
#> 3 hippocampus       (((0.82329 0.59152, 0.82025 0.58734, 0.78993 0.54558, 0.786…
#> 4 lateral ventricle (((1.64163 1.63765, 1.63383 1.63504, 1.55623 1.60883, 1.548…
#> 5 pallidum          (((0.50883 1.27848, 0.51084 1.28315, 0.53129 1.32971, 0.533…
#> 6 putamen           (((2.21213 1.27837, 2.20953 1.26015, 2.18353 1.07797, 2.180…

coronal_ref_data |> pull(region)
#> [1] "amygdala"          "caudate"           "hippocampus"      
#> [4] "lateral ventricle" "pallidum"          "putamen"          
#> [7] "thalamus proper"   "ventral DC"        NA
```

``` r
geom_aseg <- function (mapping = aes(), data = NULL, stat = ggregions::StatRegion, position = "identity", 
    na.rm = FALSE, show.legend = NA, inherit.aes = TRUE, ref_data = coronal_ref_data, 
    ...) 
{
    c(layer_sf(geom = GeomSf, data = data, mapping = mapping, 
        stat = stat, position = position, show.legend = show.legend, 
        inherit.aes = inherit.aes, params = rlang::list2(na.rm = na.rm, 
            ref_data = ref_data, ...)), coord_sf(crs = NULL))
}


stamp_brain <- function (mapping = aes(), data = ref_data, stat = ggregions:::StatRegion, 
    position = "identity", na.rm = FALSE, show.legend = NA, inherit.aes = FALSE, 
    ref_data = coronal_ref_data, ...) 
{
    c(layer_sf(geom = GeomSf, data = data, mapping = mapping, 
        stat = stat, position = position, show.legend = show.legend, 
        inherit.aes = inherit.aes, params = rlang::list2(na.rm = na.rm, 
            ref_data = ref_data, stamp = T, ...)), coord_sf(crs = NULL))
}

geom_aseg_text <- function (mapping = aes(), data = NULL, stat = ggregions:::StatRegion, position = "identity", 
    na.rm = FALSE, show.legend = NA, inherit.aes = TRUE, ref_data = coronal_ref_data, 
    ...) 
{
    c(layer_sf(geom = GeomText, data = data, mapping = mapping, 
        stat = stat, position = position, show.legend = show.legend, 
        inherit.aes = inherit.aes, params = rlang::list2(na.rm = na.rm, 
            ref_data = ref_data, ...)), coord_sf(crs = NULL))
}

stamp_brain_text <- function (mapping = aes(), data = ref_data, stat = ggregions:::StatRegion, 
    position = "identity", na.rm = FALSE, show.legend = NA, inherit.aes = FALSE, 
    ref_data = coronal_ref_data, ...) 
{
    c(layer_sf(geom = GeomText, data = data, mapping = mapping, 
        stat = stat, position = position, show.legend = show.legend, 
        inherit.aes = inherit.aes, params = rlang::list2(na.rm = na.rm, 
            ref_data = ref_data, stamp = T, ...)), coord_sf(crs = NULL))
}


library(ggseg) # branch

ggplot() + 
  stamp_brain() + 
  stamp_brain(keep = "hippocampus", fill = "blue")
#> Coordinate system already present.
#> ℹ Adding new coordinate system, which will replace the existing one.
```

<img src="man/figures/README-unnamed-chunk-3-1.png" width="100%" />

``` r

last_plot() + 
  stamp_brain(keep = "amygdala", fill = "magenta")
#> Coordinate system already present.
#> ℹ Adding new coordinate system, which will replace the existing one.
```

<img src="man/figures/README-unnamed-chunk-3-2.png" width="100%" />

``` r

last_plot() + 
  stamp_brain(keep = "brain stem", fill = "lateral ventricle")
#> Coordinate system already present.
#> ℹ Adding new coordinate system, which will replace the existing one.
#> Warning: Computation failed in `stat_region()`.
#> Caused by error in `coordinates[, "X"]`:
#> ! no 'dimnames' attribute for array
```

<img src="man/figures/README-unnamed-chunk-3-3.png" width="100%" />

``` r

ggregions:::StatRegion$compute_panel
#> <ggproto method>
#>   <Wrapper function>
#>     function (...) 
#> compute_panel(...)
#> 
#>   <Inner function (f)>
#>     function (data, scales, ref_data, keep = NULL, drop = NULL, stamp = F) 
#> {
#>     ref_data$id <- ref_data[1][[1]]
#>     if (!is.null(keep)) {
#>         ref_data <- dplyr::filter(ref_data, id %in% keep)
#>     }
#>     if (!is.null(drop)) {
#>         ref_data <- dplyr::filter(ref_data, !(id %in% drop))
#>     }
#>     ref_data <- ggplot2::StatSfCoordinates$compute_group(ggplot2::StatSf$compute_panel(ref_data, 
#>         coord = ggplot2::CoordSf), coord = ggplot2::CoordSf)
#>     if (!stamp) {
#>         dplyr::inner_join(ref_data, data)
#>     }
#>     else {
#>         ref_data
#>     }
#> }
```

# ggseg <img src="man/figures/logo.png" align="right" alt="" width="138.5" />

<!-- badges: start -->

[![R build
status](https://github.com/ggseg/ggseg/workflows/R-CMD-check/badge.svg)](https://github.com/ggseg/ggseg/actions)
[![CRAN
status](https://www.r-pkg.org/badges/version/ggseg)](https://CRAN.R-project.org/package=ggseg)
[![downloads](https://CRANlogs.r-pkg.org/badges/last-month/ggseg?color=blue)](https://r-pkg.org/pkg/ggseg)
[![codecov](https://codecov.io/gh/ggseg/ggseg/branch/main/graph/badge.svg?token=WtlS6Kk1vo)](https://app.codecov.io/gh/ggseg/ggseg)
[![Lifecycle:
maturing](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://lifecycle.r-lib.org/articles/stages.html)
[![Codecov test
coverage](https://codecov.io/gh/ggseg/ggseg/graph/badge.svg)](https://app.codecov.io/gh/ggseg/ggseg)
[![R-CMD-check](https://github.com/ggseg/ggseg/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/ggseg/ggseg/actions/workflows/R-CMD-check.yaml)
<!-- badges: end -->

Contains ggplot2 geom for plotting brain atlases using simple features.
The largest component of the package is the data for the two built-in
atlases. Plotting results of analyses on regions or networks often
involves swapping between statistical tools, like R, and software for
brain imaging to correctly visualise analysis results.

This package aims to make it possible to plot results directly through
R.

## Atlases

There are currently four atlases available in the package:

1.  `dk` - Desikan-Killiany atlas (aparc).  
2.  `aseg` - Automatic subcortical segmentation.

**Note:** As of version 1.5.3, `ggseg` was split into two packages: one
for 2d polygon plots in ggplot, and another for 3d mesh plots through
plotly. This was done to reduce package size, dependencies, and also to
simplify maintenance. If you want the 3d plotting tool, please go the
[ggseg3d repository](https://github.com/ggseg/ggseg3d).

You may find more atlases and functions to create new atlases in the
companion package [ggsegExtra](https://github.com/ggseg/ggsegExtra).

## Installation

The package can be installed from CRAN.

``` r
install.packages("ggseg")
```

Alternatively, ggseg may also be installed through its ggseg r-universe:

``` r
# Enable this universe
options(repos = c(
    ggseg = 'https://ggseg.r-universe.dev',
    CRAN = 'https://cloud.r-project.org'))

# Install some packages
install.packages('ggseg')
```

The development version of the package can be installed using devtools.

``` r
install.packages("remotes")
remotes::install_github("ggseg/ggseg")
```

The functions are now installed, and you may load them when you want to
use them. All functions are documented in standard R fashion.

## Use

``` r
library(ggseg)
library(ggplot2)
plot(dk)
```

<img src="man/figures/README-unnamed-chunk-6-1.png" width="100%" />

``` r
plot(aseg)
```

<img src="man/figures/README-unnamed-chunk-6-2.png" width="100%" />

While default atlas plots will give you an idea of how the atlases look,
you will likely want to project your own data onto the plot.

``` r
library(dplyr)
some_data <- tibble(
  region = rep(c("transverse temporal", "insula",
           "precentral","superior parietal"), 2), 
  p = sample(seq(0,.5,.001), 8),
  groups = c(rep("g1", 4), rep("g2", 4))
)

some_data |>
  group_by(groups) |>
  ggplot() +
  geom_brain(atlas = dk, 
             position = position_brain(hemi ~ side),
             aes(fill = p)) +
  facet_wrap(~groups)
#> merging atlas and data by 'region'
```

<img src="man/figures/README-unnamed-chunk-7-1.png" width="100%" />

The package also has several vignettes, to help you get started using
it. You can access it [here](https://ggseg.github.io/ggseg/)

You can also see one of the creators blog for introductions to its use
[here](https://drmowinckels.io/blog/2021-03-14-new-ggseg-with-geom/)

### Report bugs or requests

Don’t hesitate to ask for support using [github
issues](https://github.com/ggseg/ggseg/issues), or requesting new
atlases. While we would love getting help in creating new atlases, you
may also request atlases through the issues, and we will try to get to
it.

# Funding

This tool is partly funded by:

**EU Horizon 2020 Grant:** Healthy minds 0-100 years: Optimising the use
of European brain imaging cohorts (Lifebrain).

**Grant agreement number:** 732592.

**Call:** Societal challenges: Health, demographic change and well-being
