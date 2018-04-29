# Package build fails because vignette does not find child Rmd files

Minimal R package to reproduce an issue described at https://stackoverflow.com/q/50078849/4468078

# Test setup

The following tests were done on 

```
R version 3.4.4 (2018-03-15)
Platform: x86_64-pc-linux-gnu (64-bit)
Running under: Ubuntu 14.04.5 LTS
```

and RStudio 1.1.442



# Problem description

If you use Rmarkdown with a child document for your vignette file you will observer different problems:


## `R CMD build" succeeds but the child Rmd file is missing in `inst/doc`

If there is **no existing** `inst/doc` folder in your package

* `R CMD build .` succeeds to build the source package file (`vignette.test1_1.0.0.9000.tar.gz`)
* Generates a correct HTML vignette file (including the contents of child Rmd files)
* but the tar.gz file does **NOT** contain the child Rmd file `child_doc.Rmd` in
  the `inst/doc` folder (only in the `vignettes` folder)

You can verify this after installation of the package
(e. g. with `install.packages("vignette.test1_1.0.0.9000.tar.gz", repos = NULL, type = "source" )`)
via `browseVignettes(package = "vignette.test1")`

Impact:

* the generated R file in `inst/doc` is incomplete (does not contain the R code from the child Rmd files)
* the Rmd master file alone is worthless in `inst/doc`

Note: An `R CMD check vignette.test1_1.0.0.9000.tar.gz` does succeed anyhow.



## `R CMD build` fails if an empty `inst/doc` folder exists

If there is an **existing but empty** `inst/doc` folder in your package
`R CMD build .` fails with the following message:

```
* checking for file ‘./DESCRIPTION’ ... OK
* preparing ‘vignette.test1’:
* checking DESCRIPTION meta-information ... OK
* installing the package to build vignettes
      -----------------------------------
* installing *source* package ‘vignette.test1’ ...
** R
** inst
** preparing package for lazy loading
** help
No man pages found in package  ‘vignette.test1’ 
*** installing help indices
** building package indices
** installing vignettes
   ‘master_vignette.Rmd’ using ‘UTF-8’ 
Warning in readLines(if (is.character(input2)) { :
  cannot open file './child_doc.Rmd': No such file or directory
Quitting from lines 15-15 (./child_doc.Rmd) 
Error in readLines(if (is.character(input2)) { : 
  cannot open the connection
ERROR: installing vignettes failed
* removing ‘/tmp/RtmpKoDLV1/Rinst29be5f3fd48f/vignette.test1’
      -----------------------------------
ERROR: package installation failed
```

This is (at least) unexpected.



## Building the vignette manually via `devtools::build_vignettes()` makes `R CMD build` working again

Execute `devtools::build_vignettes()` in the root folder of the package.

This results in a warning

```
Building vignette.test1 vignettes
Moving master_vignette.html, master_vignette.R to inst/doc/
Copying master_vignette.Rmd to inst/doc/
Warning message:
In tools::buildVignettes(dir = pkg$path, tangle = TRUE) :
  Files named as vignettes but with no recognized vignette engine:
   ‘vignettes/child_doc.Rmd’
(Is a VignetteBuilder field missing?)
```

and the generation of the folder `inst/doc` together with the generated
vignette files (as if you had executed `R CMD build .` which is clear since
`devtools` calls `tools::buildVignettes` as `R CMD build` does too).

The impacts are the same as using `R CMD build` (described above).


# Known workarounds

## Add the vignette header to each child Rmd file

You could add a YAML header like this to each child RMD file

```R
---
title: "child doc"
author: "Vignette Author"
date: "`r Sys.Date()`"
output: rmarkdown::html_vignette
vignette: >
  %\VignetteIndexEntry{Vignette Title}
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteEncoding{UTF-8}
---
```

to make R think it is a separate vignette file.

Disadantage:

* `browseVignettes(package = "vignette.test1")` shows the child Rmd docs as if they were separate vignettes



# Current status

* Problem reported at stackoverflow (https://stackoverflow.com/q/50078849/4468078)
* Proposed work-around does work (with the described side-effects)
* No issue opened for `knitr` so far...



# Next steps

1. Open an issue for `knitr`

2. Try with the newest R version

2. If `knitr` cannot solve the problem and the up-to-date R version has not already solved the problems:
   
   Ask at r-devel if this could be improved
   


