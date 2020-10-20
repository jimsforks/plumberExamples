
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Future

Example for using the
[future](https://github.com/HenrikBengtsson/future) package for
asynchronous execution with Plumber

## Endpoints

-   `/sync`: Print route, time, and worker PID
-   `/future`: Sleep for 10 seconds, then print route, time, and worker
    PID
-   `/divide`: Divide a by b in a future process
-   `/divide-catch`: Divide a by b in a future process and handle errors

## Definition

### [plumber.R](plumber.R)

``` r
library(promises)
library(future)

future::plan("multiprocess") # use all available cores
# future::plan(future::multiprocess(workers = 2)) # only two cores

# Quick manual test:
# Within 10 seconds...
# 1. Visit /future
# 2. While /future is loading, visit /sync many times
# /future will not block /sync from being able to be loaded.


#' @serializer json list(auto_unbox = TRUE)
#' @get /sync
function() {
  # print route, time, and worker pid
  paste0("/sync; ", Sys.time(), "; pid:", Sys.getpid())
}

#' @contentType list(type = "text/html")
#' @serializer json list(auto_unbox = TRUE)
#' @get /future
function() {

  future({
    # perform large computations
    Sys.sleep(10)

    # print route, time, and worker pid
    paste0("/future; ", Sys.time(), "; pid:", Sys.getpid())
  })
}


# Originally by @antoine-sachet from https://github.com/rstudio/plumber/issues/389
#' @get /divide
#' @serializer json list(auto_unbox = TRUE)
#' @param a number
#' @param b number
function(a = NA, b = NA) {
  future({
    a <- as.numeric(a)
    b <- as.numeric(b)
    if (is.na(a)) stop("a is missing")
    if (is.na(b)) stop("b is missing")
    if (b == 0) stop("Cannot divide by 0")

    a / b
  })
}

#' @get /divide-catch
#' @serializer json list(auto_unbox = TRUE)
#' @param a number
#' @param b number
function(a = NA, b = NA) {
  future({
    a <- as.numeric(a)
    b <- as.numeric(b)
    if (is.na(a)) stop("a is missing")
    if (is.na(b)) stop("b is missing")
    if (b == 0) stop("Cannot divide by 0")

    a / b
  }) %>%
    # Handle `future` errors
    promises::catch(function(error) {
      # handle error here!
      if (error$message == "b is missing") {
        return(Inf)
      }

      # rethrow original error
      stop(error)
    })
}
```
