# Built with the help of http://kevinushey.github.io/blog/2015/02/02/rprofile-essentials/
# adapted from https://github.com/kevinushey/etc/blob/master/dotfiles/.Rprofile
# and https://github.com/jennybc/system-setup/blob/master/dotfiles/.Rprofile.R


.First <- function() {
  
  # set TZ if unset
  if (is.na(Sys.getenv("TZ", unset = NA)))
    Sys.setenv(TZ = "America/Los_Angeles")
  
  # only run in interactive mode
  if (!interactive())
    return()
  
  # bail in Docker environments
  if (!is.na(Sys.getenv("DOCKER", unset = NA)))
    return()
  
  # create .Rprofile env
  .__Rprofile.env__. <- attach(NULL, name = "local:rprofile")
  
  # Ensure that the user library exists and is set, so that we don't install to
  # the system library by default (e.g. on OS X)
  local({
    userLibs <- strsplit(Sys.getenv("R_LIBS_USER"), .Platform$path.sep)[[1]]
    if (length(userLibs) && is.character(userLibs)) {
      lapply(userLibs, function(lib) {
        if (!file.exists(lib)) {
          if (!dir.create(lib, recursive = TRUE)) {
            warning("failed to create user library '", lib, "'")
          }
        }
      })
    }
    .libPaths(userLibs)
  })
  
  # ensure Rtools on PATH for Windows
  isWindows <- Sys.info()[["sysname"]] == "Windows"
  # Ensure /usr/local/bin is first on the PATH
  if (!isWindows) {
    PATH <- strsplit(Sys.getenv("PATH"), .Platform$path.sep, fixed = TRUE)[[1]]
    PATH <- unique(c("/usr/local/bin", PATH))
    Sys.setenv(PATH = paste(PATH, collapse = .Platform$path.sep))
  }
  
  # prefer source packages for older versions of R
  pkgType <- getOption("pkgType")
  if (getRversion() < "3.3")
    pkgType <- "source"
  
  options(
    # CRAN
    repos = c(CRAN = "https://cran.rstudio.org"),
    
    # source packages for older R
    pkgType = pkgType,
    
    # no tcltk
    menu.graphics = FALSE,
    
    # don't treate newlines as synonym to 'n' in browser
    browserNLdisabled = TRUE,
    
    # keep source code as-is for package installs
    keep.source = TRUE,
    keep.source.pkgs = TRUE,
    
    ## fancy quotes are annoying and lead to
    ## 'copy + paste' bugs / frustrations
    useFancyQuotes = FALSE,
    
    # warn on partial matches
    # warnPartialMatchArgs = TRUE ## too disruptive
    warnPartialMatchAttr = TRUE,
    warnPartialMatchDollar = TRUE,
    warnPartialMatchArgs = TRUE,
    
    # warn right away
    warn = 1
    
    ## warnings are errors
    # warn = 2
  )
  
  DEVTOOLS.DESC.AUTHOR <-
    paste0("person(\"Jessica\", \"Minnier\", ",
           "role=c(\"aut\", \"cre\"),\n  email = \"minnier@ohsu.edu\")")
  
  # devtools options
  options(
    devtools.name = "Jessica Minnier",
    devtools.desc = list(
      `Authors@R`     = DEVTOOLS.DESC.AUTHOR,
      License         = "MIT + file LICENSE",
      Version         = "0.0.0.9000"
    )
  )
  #      VignetteBuilder = "knitr")
  
  rm(DEVTOOLS.DESC.AUTHOR)
  
  # always run Rcpp tests
  Sys.setenv(RunAllRcppTests = "yes")
  
  # auto-completion of package names in `require`, `library`
  utils::rc.settings(ipck = TRUE)
  
  
  # ensure commonly-used packages are installed, loaded
  quietly <- function(expr) {
    status <- FALSE
    suppressWarnings(suppressMessages(
      utils::capture.output(status <- expr)
    ))
    status
  }
  
  install <- function(package) {
    
    code <- sprintf(
      "utils::install.packages(%s, lib = %s, repos = %s)",
      shQuote(package),
      shQuote(.libPaths()[[1]]),
      shQuote(getOption("repos")[["CRAN"]])
    )
    
    R <- file.path(
      R.home("bin"),
      if (Sys.info()[["sysname"]] == "Windows") "R.exe" else "R"
    )
    
    con <- tempfile(fileext = ".R")
    writeLines(code, con = con)
    on.exit(unlink(con), add = TRUE)
    
    cmd <- paste(shQuote(R), "-f", shQuote(con))
    system(cmd, ignore.stdout = TRUE, ignore.stderr = TRUE)
  }
  
  packages <- c("devtools", "roxygen2", "knitr", "rmarkdown", "testthat")
  invisible(lapply(packages, function(package) {
    
    if (quietly(require(package, character.only = TRUE, quietly = TRUE)))
      return()
    
    message("Installing '", package, "' ... ", appendLF = FALSE)
    install(package)
    
    success <- quietly(require(package, character.only = TRUE, quietly = TRUE))
    message(if (success) "OK" else "FAIL")
    
  }))
  
  # clean up extra attached envs
  addTaskCallback(function(...) {
    count <- sum(search() == "local:rprofile")
    if (count == 0)
      return(FALSE)
    
    for (i in seq_len(count - 1))
      detach("local:rprofile")
    
    return(FALSE)
  })
  
  # display startup message(s)
  msg <- if (length(.libPaths()) > 1)
    "Using libraries at paths:\n"
  else
    "Using library at path:\n"
  libs <- paste("-", .libPaths(), collapse = "\n")
  message(msg, libs, sep = "")
}
