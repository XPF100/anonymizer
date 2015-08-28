<!-- README.md is generated from README.Rmd. Please edit that file -->
anonymizer
==========

[![Build Status](https://travis-ci.org/paulhendricks/anonymizer.png?branch=master)](https://travis-ci.org/paulhendricks/anonymizer) [![Build status](https://ci.appveyor.com/api/projects/status/qu5j8q9wvit2i3pe/branch/master?svg=true)](https://ci.appveyor.com/project/paulhendricks/anonymizer/branch/master) [![codecov.io](http://codecov.io/github/paulhendricks/anonymizer/coverage.svg?branch=master)](http://codecov.io/github/paulhendricks/anonymizer?branch=master) [![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/anonymizer)](http://cran.r-project.org/package=anonymizer) [![Downloads from the RStudio CRAN mirror](http://cranlogs.r-pkg.org/badges/anonymizer)](http://cran.rstudio.com/package=anonymizer) [![Project Status: Active - The project has reached a stable, usable state and is being actively developed.](http://www.repostatus.org/badges/0.1.0/active.svg)](http://www.repostatus.org/#active)

`anonymizer` [anonymizes](https://en.wikipedia.org/wiki/Data_anonymization) data containing [Personally Identifiable Information](https://en.wikipedia.org/wiki/Personally_identifiable_information) (PII) using a combination of [salting](https://en.wikipedia.org/wiki/Salt_%28cryptography%29) and [hashing](https://en.wikipedia.org/wiki/Hash_function). You can find quality examples of data anonymization in R [here](http://jangorecki.github.io/blog/2014-11-07/Data-Anonymization-in-R.html), [here](http://stackoverflow.com/questions/10454973/how-to-create-example-data-set-from-private-data-replacing-variable-names-and-l), and [here](http://4dpiecharts.com/2011/08/23/anonymising-data/).

Installation
------------

You can install:

-   the latest released version from CRAN with

    ``` r
    install.packages("anonymizer")
    ```

-   the latest development version from github with

    ``` r
    if (packageVersion("devtools") < 1.6) {
      install.packages("devtools")
    }
    devtools::install_github("paulhendricks/anonymizer")
    ```

If you encounter a clear bug, please file a minimal reproducible example on [github](https://github.com/paulhendricks/anonymizer/issues).

API
---

`anonymzer` employs six convenience functions: `salt`, `unsalt`, `hash`, and `anonymize`.

``` r
library(dplyr, warn.conflicts = FALSE)
library(anonymizer)
letters %>% head
#> [1] "a" "b" "c" "d" "e" "f"
letters %>% head %>% salt(.seed = 1)
#> [1] "gjoxfagjoxf" "gjoxfbgjoxf" "gjoxfcgjoxf" "gjoxfdgjoxf" "gjoxfegjoxf"
#> [6] "gjoxffgjoxf"
letters %>% head %>% salt(.seed = 1) %>% unsalt(.seed = 1)
#> [1] 1
letters %>% head %>% hash(.algo = "crc32", .seed = 1)
#> [1] "c0749952" "597dc8e8" "2e7af87e" "b01e6ddd" "c7195d4b" "5e100cf1"
letters %>% head %>% salt(.seed = 1) %>% hash(.algo = "crc32", .seed = 1)
#> [1] "b0891ad8" "361d6876" "fd41bbd3" "e0448b6b" "2b1858ce" "ad8c2a60"
letters %>% head %>% anonymize(.algo = "crc32", .seed = 1)
#> [1] "b0891ad8" "361d6876" "fd41bbd3" "e0448b6b" "2b1858ce" "ad8c2a60"
```

### Generate data containing fake PII

``` r
library(generator)
set.seed(1)
n <- 6
ashley_madison <- 
  data.frame(name = r_full_names(n), 
             email = r_email_addresses(n), 
             phone_number = r_phone_numbers(n, use_hyphens = TRUE, 
                                            use_spaces = TRUE), 
             stringsAsFactors = FALSE)
ashley_madison %>% 
  knitr::kable(format = "markdown")
```

| name                  | email                     | phone\_number  |
|:----------------------|:--------------------------|:---------------|
| Eldridge Pfannerstill | <yfpc@gjaithyl.met>       | 421- 714- 6843 |
| Augustine Homenick    | <csjyoql@ntakqojv.lgb>    | 274- 528- 6517 |
| Jennie Runte          | <n@rkvg.lip>              | 685- 657- 5692 |
| Araceli Kunde         | <tcv@iuzhk.xvj>           | 364- 261- 8542 |
| Josue Rau             | <lrjhqeoc@gdfbosptiz.vpy> | 592- 512- 2378 |
| Elnora Zemlak         | <zpfc@mxo.zsi>            | 893- 136- 4687 |

### Detect data containing PII

``` r
library(detector)
ashley_madison %>% 
  detect %>% 
  knitr::kable(format = "markdown")
```

| column\_name  | has\_email\_addresses | has\_phone\_numbers | has\_national\_identification\_numbers |
|:--------------|:----------------------|:--------------------|:---------------------------------------|
| name          | FALSE                 | FALSE               | FALSE                                  |
| email         | TRUE                  | FALSE               | FALSE                                  |
| phone\_number | FALSE                 | TRUE                | FALSE                                  |

### Anonymize data containing PII

``` r
ashley_madison %>% 
  mutate(name = anonymize(name, .algo = "crc32", .seed = 1), 
         email = anonymize(email, .algo = "crc32", .seed = 1), 
         phone_number = anonymize(phone_number, .algo = "crc32", .seed = 1)) %>% 
  knitr::kable(format = "markdown")
```

| name     | email    | phone\_number |
|:---------|:---------|:--------------|
| 6e557a48 | bba0f582 | 2d3a47ee      |
| e818a264 | 518f7f6  | abb05c14      |
| 685ed40  | afc10b09 | 94a766dd      |
| c335a390 | 4c48c731 | 61eaf5ec      |
| 4366c250 | 9a7fa137 | 16f2915       |
| cc7830c7 | c7e0516  | f7f0c694      |
