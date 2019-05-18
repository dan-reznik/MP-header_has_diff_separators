Read a csv whose first line has different separators
================

Load libraries

``` r
library(tidyverse)
library(fs)
```

List contents of data directory

``` r
dir_ls("data")
#> data/wrong_header.csv
```

Read first 2 lines of file, noticing header with incorrect separators

``` r
fname1 <- "data/wrong_header.csv"
read_lines(fname1,n_max=2)
#> [1] "name|birthdate|height|kgs"  "danno;2001-05-22;1,73;75,4"
```

Fix header

``` r
fixed_header <- read_lines(fname1,n_max=1) %>%
  str_replace_all(fixed("|"),";")
fixed_header
#> [1] "name;birthdate;height;kgs"
```

Concatenate fixed header with body of file (without the original header)
and pipe result to read\_csv2

``` r
df_sensei <- c(fixed_header,read_lines(fname1,skip=1)) %>%
  str_c(collapse="\n") %>%
  read_csv2
#> Using ',' as decimal and '.' as grouping mark. Use read_delim() for more control.
```

| name   | birthdate  | height |  kgs |
| :----- | :--------- | -----: | ---: |
| danno  | 2001-05-22 |   1.73 | 75.4 |
| manno  | 2002-06-23 |   1.83 | 85.4 |
| weirdo | 2003-07-24 |   1.93 | 91.3 |

-----

Never give up. Every frustration is a necessary step in the path toward
mastery.

<img src="pics/ippon.gif" width="25%" style="display: block; margin: auto auto auto 0;" />
