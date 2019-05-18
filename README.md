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

Read first lines of file, noticing header with incorrect separators

``` r
fname1 <- "data/wrong_header.csv"
read_lines(fname1,n_max=3)
#> [1] "name|birthdate|height|kgs"  "danno;2001-05-22;1,73;75,4"
#> [3] "manno;2002-06-23;1,83;85,4"
```

Let us fix the separators in the header

``` r
fixed_header <- read_lines(fname1,n_max=1) %>%
  str_replace_all(fixed("|"),";")
fixed_header
#> [1] "name;birthdate;height;kgs"
```

# Solution 1: concatenate fixed header and body

Concatenate fixed header with body of file (without the original header)
and pipe result to read\_csv2

``` r
df_sensei1 <- c(fixed_header,
                read_lines(fname1,skip=1)) %>%
  str_c(collapse="\n") %>%
  read_csv2
```

| name   | birthdate  | height |  kgs |
| :----- | :--------- | -----: | ---: |
| danno  | 2001-05-22 |   1.73 | 75.4 |
| manno  | 2002-06-23 |   1.83 | 85.4 |
| weirdo | 2003-07-24 |   1.93 | 91.3 |

# Solution 2: replace wrong separators on first line in entire file

warning: make sure file does not contain any other “|”

``` r
df_sensei2 <- read_file(fname1) %>%
  str_replace_all(fixed("|"),";") %>%
  read_csv2
```

| name   | birthdate  | height |  kgs |
| :----- | :--------- | -----: | ---: |
| danno  | 2001-05-22 |   1.73 | 75.4 |
| manno  | 2002-06-23 |   1.83 | 85.4 |
| weirdo | 2003-07-24 |   1.93 | 91.3 |

# Solution 3: combine fixed header with original file, skipping header chars

If we skip the body of the file by the length of the fixed\_header + 2
we get the entire file after the first line:

``` r
read_file(fname1) %>% str_sub(start=2+str_length(fixed_header))
#> [1] "danno;2001-05-22;1,73;75,4\nmanno;2002-06-23;1,83;85,4\nweirdo;2003-07-24;1,93;91,3\n"
```

So we can simply combine fixed header and remainder of the file:

``` r
skip_chars <- function(s,skip) str_sub(s,start=skip)

skip <- 2+str_length(fixed_header)
df_sensei3 <- c(fixed_header,
                skip_chars(read_file(fname1),skip)) %>%
  str_c(collapse="\n") %>%
  read_csv2
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
