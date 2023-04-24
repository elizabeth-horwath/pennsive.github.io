# R Packages

## Making R Packages

When writing a methodological paper, it is a good idea to make your code publicly available, since it will be much easier for others to use your method on their own data. One way to do this is to create an R package.

R packages can be complicated, but the easiest way to get started is to make a **bare-bones, minimal R package** and then add complexity. This article will show you how!

**Note**: These instructions are blatantly stolen from  [Hilary Parker's article](https://hilaryparker.com/2014/04/29/writing-an-r-package-from-scratch/), [Karl Broman's article](https://kbroman.org/pkg_primer/), [Andrew's slides](https://andy1764.github.io/Making%20an%20R%20Package/Making-an-R-Package.html), and Danni's teaching materials from BSTA 670. Everything was tested on R version 4.1.3.

## TLDR; You Can Make an R Package with 3 Lines of Code

- `devtools::create()`
- `devtools::document()`
- `devtools::install()`

## Example: `pennsive_stuff`

Our goal is to make a minimal package called `pennsive_stuff`, consisting of 2 R functions. Suppose these functions are located in two `.R` files, `nl.R` and `plot_dist.R`, containing the following:

- `nl()`: a function that prints new lines in R Markdown documents.

```
# Contents of nl.R
nl <- function(){
  cat("  \n")
  cat("  \n")
}
```

- `plot_dist()`: a function that plots the distribution of a variable in a dataset.

```
# Contents of plot_dist.R
plot_dist <- function(dat, var_name){
  stopifnot(var_name %in% colnames(dat))
  stopifnot(class(dat[,var_name]) == "numeric")
  
  p <- ggplot2::ggplot(dat) +
    ggplot2::geom_histogram(ggplot2::aes_string(x = var_name), bins = 30, 
                            fill = wesanderson::wes_palettes$GrandBudapest2[1]) +
    ggplot2::theme_minimal() +
    ggplot2::labs(y = "Count", x = var_name)
  
  return(p)
}
```

Note that `plot_dist()` contains dependencies from the `ggplot` and `wesanderson` packages.

### 1. Install R packages to help you make packages

```
# Run in R
install.packages(c("available", "usethis", "testthat", "pkgdown", "roxygen2", "devtools", "covr"))
```

(But how were *those* packages created?? Ooooo!)

To make a super minimal package, you really only need `roxygen2` and `devtools`.

### 1a. (Optional) Check that your package name is valid

This will also check if the name has been taken on CRAN and other sites.

```
# Run in R
# Will look up your package name on CRAN, Bioconductor, and Github
# Will also check the meaning of your package name on several websites
available::available(name = "pennsive_stuff", browse = FALSE)
```

### 2. Create Package Directory

```
# Run in R
dir.package <- [the directory on your computer that you'd like everything to be in]
setwd(dir.package)
devtools::create("pennsive_stuff")
```

This will automatically create a folder in your working directory called `pennsive_stuff`:

```
# Run in R
list.files("pennsive_stuff")
```

- `DESCRIPTION` has details about your package, which can be edited in an app like TextEdit (Mac) or WordPad (Windows).
- `NAMESPACE` has the names of the objects in your package that will be made available upon `library()`ing it. It's empty right now!
- `R` is a folder that will hold all of your function `.R` files.

### 3. Add Functions to the R Folder

Each function that you want to include in the package should be in an `.R` file and placed in the folder `pennsive_stuff/R` that was created in Step 2. You can have one function per `.R` file, or organize multiple functions in one `.R` file. It doesn't mattter! 

Let's put the `.R` files, `nl.R` and `plot_dist.R`--which contain the code for `nl()` and `plot_dist()` respectively--into `pennsive_stuff/R`:

```
# Run in R. 
# Ensure that the two files, nl.R and plot_dist.R are in pennsive_stuff/R
list.files(file.path("pennsive_stuff", "R"))
```

### 4. Write Documentation for Functions

You know the nice notes that you see when you do e.g. `?sample`? We too can have nice things!

The most tedious parts of documentation are taken care of through the `Roxygen2` package. All you have to do is add special comments at the start of the `.R` files that you just copied to `/pennsive_stuff/R`.

Roxygen2 documentation is just extra lines added before the functions in the `.R` files and they look roughly like this:

```
#' Short summary of the function
#'
#' Description of the function
#'
#' @param param_name1 description of parameter/input 1
#' @param param_name1 description of parameter/input 1
#'
#' @return description of what is returned
#'
#' @examples
#' nl()
#'
#' @export
```

- The final line `@export` adds this function to the NAMESPACE file, so that is accessible (i.e., via `pennsive_stuff::nl()` or after calling library). Otherwise, you would have to access it through the `:::` operator, à la `pennsive_stuff:::nl()`.
- There are many options for formatting these things (e.g., adding links, equations): see [https://roxygen2.r-lib.org/articles/formatting.html](https://roxygen2.r-lib.org/articles/formatting.html).

For example, after adding documentation to `nl.R`, the file would look like:

```
# Contents of nl.R

#' Print new line in HTML files
#'
#' This function prints a new line in HTML files, which is useful when printing other things such as headers or plots.
#'
#' @return Returns nothing except some spaces!!
#'
#' @examples
#' nl()
#'
#' @export

nl <- function(){
  cat("  \n")
  cat("  \n")
}
```

For `plot_dist.R`, because the function depends on the `ggplot2` and `wesanderson` R packages, we would have an additional line for `@import` (imports everything in the namespace) and `@importFrom` (imports only a particular thing).

```
# Contents of plot_dist.R

#' Plot Histogram
#'
#' Plot the histogram of a numeric variable within a data frame
#'
#' @param dat a data.frame containing the data to plot.
#' @param var_name a character vector of the column name in the data.
#'
#' @return a histogram plot.
#' @import ggplot2
#' @importFrom wesanderson wes_palettes
#'
#' @examples
#' plot_dist(dat = iris, var_name = "Petal.Length")
#'
#' @export

plot_dist <- function(dat, var_name){
  stopifnot(var_name %in% colnames(dat))
  stopifnot(class(dat[,var_name]) == "numeric")
  
  p <- ggplot2::ggplot(dat) +
    ggplot2::geom_histogram(ggplot2::aes_string(x = var_name), bins = 30, 
                            fill = wesanderson::wes_palettes$GrandBudapest2[1]) +
    ggplot2::theme_minimal() +
    ggplot2::labs(y = "Count", x = var_name)
  
  return(p)
}
```

#### 4a. Process Documentation

```
# Run in R
setwd(file.path(dir.package, "pennsive_stuff"))
devtools::document()
```

After running `document()`, you will see some `.Rd` files automatically generated for you in the `man` folder:

```
# Run in R
list.files("man")
```

### 5. Install Package

That was it! Easy, right? Now let's install the package while in the directory containing the `pennsive_stuff` folder:

```
# Run in R
setwd(dir.package)
devtools::install("pennsive_stuff")
```

Now you have the package!! You can call the `library` command from anywhere (on your computer) now.

```
# Run in R
library(pennsive_stuff)
nl()
plot_dist(dat = iris, var_name = "Petal.Length")
```

And you can see the results of the `Roxygen2` documentation here:

```
# Run in R
?nl()
?plot_dist()
```

Great job!

## Extras

You have a basic R package; everything else is gravy. Here are some more things you can do with your package:

- Include data.
- Upload to Github, which allows your package to be installed via `devtools::install_github()`.
- Uplaod to CRAN or Bioconductor.

## Extras 1: Including Data

Some packages come with data (e.g. `refund::DTI`) that accompany the functions. Suppose we also want the package to contain the dataset `iris2`.

In the package directory, create a folder called `data`:

```
# Run in R
if (!dir.exists(file.path(dir.package, "pennsive_stuff/data"))) dir.create(file.path(dir.package, "pennsive_stuff/data"))
```

The easiest way to save the data in the `data` folder is to use `usethis::usedata()`:

```
# Run in R
set.seed(382349)
iris2 = iris[sample(x = 1:nrow(iris), size = 100, replace = TRUE),]

setwd(file.path(dir.package, "pennsive_stuff"))
usethis::use_data(iris2, iris2)
```

Then, you'll need to create an R file named `data.R` in the `R` folder, where your other R functions are stored.

Similar to the `Roxygen2` documentation for functions, the `data.R` file needs to contain the following:

```
# Contents of data.R

#' Edgar Anderson's Iris Data
#'
#' This famous (Fisher's or Anderson's) iris data set gives the measurements in 
#' centimeters of the variables sepal length and width and petal length and width, 
#' respectively, for 50 flowers from each of 3 species of iris. The species are 
#' Iris setosa, versicolor, and virginica.
#'
#' @docType data
#'
#' @usage data(iris2)
#'
#' @format \code{"iris"} is a data frame with 150 cases (rows) and 5 variables
#'  (columns) named Sepal.Length, Sepal.Width, Petal.Length, Petal.Width, and Species.
#'
#' @keywords datasets
#'
#' @references Becker, R. A., Chambers, J. M. and Wilks, A. R. (1988) 
#' The New S Language. Wadsworth & Brooks/Cole. 

#'
#' @source \href{https://phenome.jax.org/projects/Moore1b}{QTL Archive}
#'
#' @examples
#' data(iris2)
#' plot_dist(dat = iris2, var_name = "Petal.Width")
"iris2"
```

Now you can re-document and re-install the package and get the `iris2` data using `data(iris2)`!

```
# Run in R

devtools::document()

setwd(file.path(dir.package))
devtools::install("pennsive_stuff")
library(pennsive_stuff)

data(iris2)
plot_dist(dat = iris2, var_name = "Sepal.Length")
?iris2
```

## Extras 2: Version Control

Keep track of the versions of your package using Git. Once you set up a Git repo in your local drive, it's easy to share it on Github (as easy as anything with Git/Github, anyways...)

#### E2.1. Create git repo in package folder

First, `cd` to the package folder `/pennsive_stuff/` in your Terminal and type

```
# Run in terminal
git init
git add .
git commit
```

If you are using RStudio and have opened the project `pennsive_stuff.Rproj`, you should see a "Git" tab in the Environment, History, Connections tab.

#### E2.2. Create repo on Github

Next, go to Github and make a new repo with the name as this package, `pennsive_stuff`. Leave the box for README unchecked!

You may need to configure RStudio access to your Github account via a personal access token. Easy instructions here: [https://gist.github.com/Z3tt/3dab3535007acf108391649766409421](https://gist.github.com/Z3tt/3dab3535007acf108391649766409421).

#### E2.3. Connect repo in package folder to github

In terminal,

```{r}
# Run in terminal
git remote add origin https://github.com/[your github-id]/pennsive_stuff
git push origin HEAD:main
```

For more details, see [https://andy1764.github.io/Making%20an%20R%20Package/Making-an-R-Package.html#47](https://andy1764.github.io/Making%20an%20R%20Package/Making-an-R-Package.html#47) and [https://kbroman.org/pkg_primer/pages/github.html](https://kbroman.org/pkg_primer/pages/github.html).


## Extras 3: Uploading to CRAN/Bioconductor

There are many hoops to jump through to put your package on CRAN. CRAN is maintained by volunteers, who manually check your R package (especially if it's the first submission) in addition to all of the automated checks. [https://kbroman.org/pkg_primer/pages/cran.html](https://kbroman.org/pkg_primer/pages/cran.html).

You can also put your package on Bioconductor, which is reportedly easier: [https://bioconductor.org/developers/package-submission/](https://bioconductor.org/developers/package-submission/).


<!--To view the wiki, cd to pennsive.github.io and run: bash run.sh -->