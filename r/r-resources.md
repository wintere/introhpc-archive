---
title: Additional R Resources
layout: page
permalink: /r-talapas/resources.html
parent: R on Talapas
nav_enabled: true
nav_order: 3
---

# Additional R Resources

## R for Computational Reserachers
-  [**Reproducibility in R and RStudio**](https://unc-libraries-data.github.io/R-Open-Labs/week2_Workflow/R_OpenLabs_Workflow.html) A lesson on reproducibility in R and Rstudio from UNC's BeginR curriculum.
- [**R for Data Science**](https://r4ds.had.co.nz/index.html) A free book on data analysis in R using [**Tidyverse**](https://www.tidyverse.org/).

## R Parallel Computing Packages
CRAN has an extensive list of [**parallel computing**](https://cran.r-project.org/web/views/HighPerformanceComputing.html) packages. 
We've tested a few.

- [**R Futures**](https://cran.r-project.org/web/packages/future/vignettes/future-1-overview.html) An R API for asynchronous programming compatible with Slurm.
- [**parallel**](https://r-universe.dev/manuals/parallel.html) This built-in R library will behave differently on Windows than on Linux machines like Talapas compute nodes.
- [**foreach**](https://cran.r-project.org/web/packages/foreach/vignettes/foreach.html) This package can be used in combination with [**doparallel**](https://cran.r-project.org/web/packages/doFuture/vignettes/doFuture-2-dopar.html) or [**doFuture**](https://cran.r-project.org/web/packages/doFuture/vignettes/doFuture-3-dofuture.html) backends to run flexible multi-core R jobs using special for-loop constructs.

