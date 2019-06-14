Read a csv whose first line has different separators
================

We are given a pseudo-csv file where the header line has delimiters
which are different from those in the remainder lines. A text file
called `wrong_header.csv` has been placed in the “data” directory with
the following contents:

    name|birthdate|height|kgs
    danno;2001-05-22;1,73;75,4
    manno;2002-06-23;1,83;85,4
    weirdo;2003-07-24;1,93;91,3

Below we show several methods on how to read this file into a data
frame.

# Load libraries

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

# Defective header in single file

## Solution 1: concatenate fixed header and body

Concatenate fixed header with body of file (without the original header)
and pipe result to read\_csv2

``` r
df_sensei1 <- c(fixed_header,
                read_lines(fname1,skip=1)) %>%
  str_c(collapse="\n") %>%
  read_csv2
df_sensei1
```

| name   | birthdate  | height |  kgs |
| :----- | :--------- | -----: | ---: |
| danno  | 2001-05-22 |   1.73 | 75.4 |
| manno  | 2002-06-23 |   1.83 | 85.4 |
| weirdo | 2003-07-24 |   1.93 | 91.3 |

## Solution 2: replace wrong separators on first line in entire file

warning: make sure file does not contain any other “|”

``` r
df_sensei2 <- read_file(fname1) %>%
  str_replace_all(fixed("|"),";") %>%
  read_csv2
df_sensei2
```

| name   | birthdate  | height |  kgs |
| :----- | :--------- | -----: | ---: |
| danno  | 2001-05-22 |   1.73 | 75.4 |
| manno  | 2002-06-23 |   1.83 | 85.4 |
| weirdo | 2003-07-24 |   1.93 | 91.3 |

## Solution 3: combine fixed header with original file, skipping header chars

If we skip the body of the file by the length of the fixed\_header + 1
(for the new line) we get the entire file after the first line:

``` r
skip_chars <- function(s,skip) str_sub(s,start=skip+1)
skip <- 1+str_length(fixed_header)

# just a test
read_file(fname1) %>% skip_chars(skip)
#> [1] "danno;2001-05-22;1,73;75,4\nmanno;2002-06-23;1,83;85,4\nweirdo;2003-07-24;1,93;91,3\n"
```

Combine fixed header and remainder of the file:

``` r
df_sensei3 <- c(fixed_header,
                skip_chars(read_file(fname1),skip)) %>%
  str_c(collapse="\n") %>%
  read_csv2
df_sensei3
```

| name   | birthdate  | height |  kgs |
| :----- | :--------- | -----: | ---: |
| danno  | 2001-05-22 |   1.73 | 75.4 |
| manno  | 2002-06-23 |   1.83 | 85.4 |
| weirdo | 2003-07-24 |   1.93 | 91.3 |

## Solution 4: low-level “seek” to skip+1

Most efficient as skipping is done on disk using `seek()`. For
black-belts only\! Note: may not work on Windows.

``` r
skip_n_read <- function(fname,skip) {
  con <- file(fname1,open="r")
  seek(con,where=skip,rw="read")
  size <- file.size(fname1)
  chars <- readChar(con,nchars=size)
  close(con)
  chars
}

df_sensei4 <- c(fixed_header,
                skip_n_read(fname1,skip)) %>%
  str_c(collapse="\n") %>%
  read_csv2
df_sensei4
```

| name   | birthdate  | height |  kgs |
| :----- | :--------- | -----: | ---: |
| danno  | 2001-05-22 |   1.73 | 75.4 |
| manno  | 2002-06-23 |   1.83 | 85.4 |
| weirdo | 2003-07-24 |   1.93 | 91.3 |

# Several files have the same defective header

In the directory “data\_many” I placed 3 slightly modified copies of the
original file with the problematic header:

``` r
fnames <- fs::dir_ls("data_many")
fnames
#> data_many/wrong_header_01.csv data_many/wrong_header_02.csv 
#> data_many/wrong_header_03.csv
```

Let’s build a table showing in each line the name of the file, the first
line, and the 2nd line:

``` r
tibble(fname=fnames,
       first_line=map_chr(fname,read_lines,n_max=1),
       second_line=map_chr(fname,read_lines,skip=1,n_max=1)) %>%
  knitr::kable()
```

| fname                            | first\_line               | second\_line                |
| :------------------------------- | :------------------------ | :-------------------------- |
| data\_many/wrong\_header\_01.csv | name|birthdate|height|kgs | danno1;2001-05-22;1,73;75,4 |
| data\_many/wrong\_header\_02.csv | name|birthdate|height|kgs | danno2;2001-05-22;1,73;75,4 |
| data\_many/wrong\_header\_03.csv | name|birthdate|height|kgs | danno3;2001-05-22;1,73;75,4 |

As before let us fix the separators in the header for the first file in
the series

``` r
fixed_header <- read_lines(fnames[1],n_max=1) %>%
  str_replace_all(fixed("|"),";")
fixed_header
#> [1] "name;birthdate;height;kgs"
```

Now let’s read all files with the fixed header and concatenate them in a
single data frame:

``` r
repair_and_read <- function(fname,fixed_header) {
  c(fixed_header,read_lines(fname,skip=1)) %>%
  str_c(collapse="\n") %>%
    read_csv2
}

df_combo <- fnames %>% map_dfr(repair_and_read,fixed_header)
```

| name    | birthdate  | height |  kgs |
| :------ | :--------- | -----: | ---: |
| danno1  | 2001-05-22 |   1.73 | 75.4 |
| manno1  | 2002-06-23 |   1.83 | 85.4 |
| weirdo1 | 2003-07-24 |   1.93 | 91.3 |
| danno2  | 2001-05-22 |   1.73 | 75.4 |
| manno2  | 2002-06-23 |   1.83 | 85.4 |
| weirdo2 | 2003-07-24 |   1.93 | 91.3 |
| danno3  | 2001-05-22 |   1.73 | 75.4 |
| manno3  | 2002-06-23 |   1.83 | 85.4 |
| weirdo3 | 2003-07-24 |   1.93 | 91.3 |

-----

Never give up. Every frustration is a necessary step in the path toward
mastery.

<img src="pics/ippon.gif" width="25%" style="display: block; margin: auto auto auto 0;" />
