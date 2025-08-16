# R Basic Tutorial
### *若需本指南的英文版本，請參閱 **[中文版本](./Tutorials/README_CN.md)**.*
---

Author: Charlene  
Version: v1.0  
Last Updated: 2025-08-14  

> This tutorial is designed for beginners in R, covering installation, syntax, data structures, data import and cleaning, visualization, basic statistics, reproducible research, and more. All examples can be directly run in base R or RStudio.  

---

## Table of Contents
- [1. Installation and Environment Setup](#1-installation-and-environment-setup)  
- [2. Core Concepts of R](#2-core-concepts-of-r)  
- [3. Basic Syntax and Objects](#3-basic-syntax-and-objects)  
- [4. Data Structures](#4-data-structures)  
- [5. Data Input/Output (I/O)](#5-data-inputoutput-io)  
- [6. Data Manipulation: base R and tidyverse](#6-data-manipulation-base-r-and-tidyverse)  
- [7. Visualization: base plot and ggplot2](#7-visualization-base-plot-and-ggplot2)  
- [8. Control Flow and Custom Functions](#8-control-flow-and-custom-functions)  
- [9. Apply Family and purrr](#9-apply-family-and-purrr)  
- [10. Basic Statistical Analysis](#10-basic-statistical-analysis)  
- [11. Debugging, Documentation, and Help](#11-debugging-documentation-and-help)  
- [12. Reproducible Research and Project Structure](#12-reproducible-research-and-project-structure)  
- [13. Common Pitfalls and Tips](#13-common-pitfalls-and-tips)  
- [14. Mini Exercises](#14-mini-exercises)  
- [Appendix A: Function Cheat Sheet](#appendix-a-function-cheat-sheet)  

---

## 1. Installation and Environment Setup
- **Install R**: Use the latest stable release (≥ 4.3).  
- **RStudio (optional)**: Provides a user-friendly IDE.  
- **Package Management**:  
  - Install packages:  
    ```r
    install.packages("tidyverse")   # dplyr, ggplot2, readr, tidyr, etc.
    install.packages("readxl")      # read Excel
    install.packages("writexl")     # write Excel
    install.packages("here")        # manage project paths
    install.packages("janitor")     # clean column names / data quality checks
    ```
  - Load packages:  
    ```r
    library(tidyverse)
    library(readxl); library(writexl)
    library(here);   library(janitor)
    ```
- **Project-Oriented Workflow**: Create an R project (RStudio: `File > New Project`) and manage relative paths with `here::here()` instead of `setwd()`.  

---

## 2. Core Concepts of R
- **Vectorized Language**: Many operations are automatically performed at the vector level.  
- **Lightweight OOP**: Common classes include `numeric`, `character`, `logical`, `factor`, `list`, `data.frame`.  
- **Assignment**:  
  ```r
  x <- 1 + 2    # preferred
  y = 3 + 4     # also valid
  x; y
  ```  
- **Pipes**: R 4.1+ includes base pipe `|>`; tidyverse also uses `%>%`.  
  ```r
  1:5 |> sum()
  # equivalent to
  sum(1:5)
  ```  

---

## 3. Basic Syntax and Objects
### 3.1 Basic Types
```r
a <- 1L           # integer
b <- 3.14         # numeric/double
c <- "hello"      # character
d <- TRUE         # logical
e <- as.factor(c("A","B","A"))  # factor
typeof(a); class(a)
```

### 3.2 Operations and Comparisons
```r
x <- c(1, 2, 3)
y <- c(10, 20, 30)
x + y          # vector addition
x * 2          # scalar multiplication
x^2            # element-wise square
x > 1          # comparison, returns logical vector
```

### 3.3 Missing Values NA
```r
z <- c(1, NA, 3)
mean(z)              # NA
mean(z, na.rm = TRUE)  # 2
is.na(z)             # check
```

### 3.4 String Processing (basic)
```r
paste("R", "rocks")       # "R rocks"
paste0("id_", 1:3)        # "id_1" "id_2" "id_3"
nchar("Data Science")     # number of characters
toupper("abc"); tolower("AbC")
```

---

## 4. Data Structures
### 4.1 Vectors
```r
v <- c(3,1,4,1,5)
v[1]          # first element
v[2:4]        # 2nd to 4th
v[v > 2]      # conditional filter
sort(v); unique(v); length(v)
```

### 4.2 Matrix and Array
```r
m <- matrix(1:6, nrow = 2, byrow = TRUE)
m[1, 2]       # row 1 col 2
dim(m)        # dimensions
```

### 4.3 List
```r
lst <- list(num = 1:3, ch = c("a","b"), flag = TRUE)
lst$num
lst[["ch"]]
```

### 4.4 Data Frame and Tibble
```r
df <- data.frame(id = 1:3, grp = c("A","B","A"), val = c(10, 20, 30))
df$val
df[ df$grp == "A", ]
tibble_df <- tibble::tibble(id = 1:3, grp = c("A","B","A"), val = c(10,20,30))
tibble_df
```

### 4.5 Factor
```r
f <- factor(c("low","high","medium"), levels = c("low","medium","high"), ordered = TRUE)
f
```

---

## 5. Data Input/Output (I/O)
### 5.1 CSV / TSV
```r
library(readr)
dat <- read_csv("data/input.csv")       # auto-detect column types
write_csv(dat, "output/result.csv")
```

### 5.2 Excel
```r
library(readxl); library(writexl)
excel_dat <- read_excel("data/input.xlsx", sheet = 1)
write_xlsx(excel_dat, "output/result.xlsx")
```

### 5.3 RDS (R native serialization)
```r
dat <- readRDS("output/dat.rds")
saveRDS(dat, "output/dat.rds")
```

### 5.4 Relative Path (recommended)
```r
library(here)
read_csv(here("data", "input.csv"))
```

> Recommended project organization: put raw data in `data/`, scripts in `R/`, outputs in `output/`.  

---

## 6. Data Manipulation: base R and tidyverse
Examples using built-in datasets `mtcars` and `iris`.  

### 6.1 Common base R operations
```r
head(mtcars, 3)
summary(mtcars)
mtcars$am <- factor(mtcars$am, labels = c("auto","manual"))
mtcars[mtcars$mpg > 25, c("mpg","hp","wt")]
```

### 6.2 dplyr Verbs
```r
library(dplyr)

mtcars |>
  as_tibble(rownames = "model") |>
  mutate(am = factor(am, labels = c("auto","manual"))) |>
  select(model, mpg, hp, wt, am) |>
  filter(mpg > 20, wt < 3.2) |>
  arrange(desc(mpg)) |>
  group_by(am) |>
  summarise(n = n(),
            mpg_mean = mean(mpg), 
            hp_median = median(hp),
            .groups = "drop")
```

### 6.3 Add/Transform Columns
```r
iris |>
  as_tibble() |>
  mutate(Petal.Ratio = Petal.Length / Petal.Width,
         Species = as.character(Species)) |>
  slice_head(n = 5)
```

### 6.4 Wide-Long Conversion (tidyr)
```r
library(tidyr)

wide <- tibble(id = 1:3, A = c(10,20,30), B = c(5,6,7))
long <- wide |>
  pivot_longer(cols = A:B, names_to = "key", values_to = "value")

restored <- long |>
  pivot_wider(names_from = key, values_from = value)
```

### 6.5 Join
```r
A <- tibble(id = 1:3, valA = c(10,20,30))
B <- tibble(id = c(2,3,4), valB = c(200,300,400))

A |> left_join(B, by = "id")
A |> inner_join(B, by = "id")
A |> full_join(B,  by = "id")
```

### 6.6 Clean Column Names (janitor)
```r
library(janitor)
dirty <- tibble("First Name" = c("Amy","Ben"), "Score(%)" = c(90, 85))
clean_names(dirty)   # "first_name", "score"
```

---

## 7. Visualization: base plot and ggplot2
### 7.1 Base R Plot
```r
plot(mtcars$wt, mtcars$mpg,
     main = "MPG vs Weight", xlab = "Weight (1000 lbs)", ylab = "MPG")
abline(lm(mpg ~ wt, data = mtcars), lwd = 2)
```

### 7.2 ggplot2 Basics
```r
library(ggplot2)

ggplot(mtcars, aes(x = wt, y = mpg, color = factor(am))) +
  geom_point(size = 2) +
  geom_smooth(method = "lm", se = FALSE) +
  labs(title = "MPG vs Weight by Transmission",
       x = "Weight (1000 lbs)",
       y = "Miles per Gallon",
       color = "Transmission") +
  theme_minimal()
```

### 7.3 Facets and Save Plots
```r
p <- ggplot(iris, aes(Sepal.Length, Sepal.Width)) +
  geom_point(alpha = 0.7) +
  facet_wrap(~ Species) +
  labs(title = "Iris by Species")
p

ggsave("output/iris_facets.png", p, width = 6, height = 4, dpi = 300)
```

---

## 8. Control Flow and Custom Functions
### 8.1 Conditionals and Loops
```r
x <- 5
if (x > 3) {
  msg <- "big"
} else {
  msg <- "small"
}

for (i in 1:3) {
  print(i^2)
}

i <- 1
while (i <= 3) {
  print(i)
  i <- i + 1
}
```

### 8.2 Custom Functions with Defaults
```r
add_and_scale <- function(x, y = 0, scale = 1) {
  stopifnot(is.numeric(x), is.numeric(y), is.numeric(scale))
  (x + y) * scale
}

add_and_scale(1:3, y = 2, scale = 0.5)
```

### 8.3 Scope and Return
```r
f <- function(x) {
  y <- x + 1      # local variable
  return(y * 2)   # explicit return
}
f(10)
```

---

## 9. Apply Family and purrr
### 9.1 base R apply Family
```r
m <- matrix(1:9, nrow = 3)
apply(m, 1, sum)    # row sums
apply(m, 2, mean)   # column means

l <- list(a = 1:3, b = 4:6)
lapply(l, mean)     # returns list
sapply(l, mean)     # simplifies output
```

### 9.2 purrr (tidyverse functional tools)
```r
library(purrr)

list(1:3, 4:6) |>
  map_dbl(mean)       # return double

# apply function to each column
iris |>
  as_tibble() |>
  map_if(is.numeric, mean) |>    # select numeric cols
  unlist()
```

---

## 10. Basic Statistical Analysis
### 10.1 Descriptive Statistics
```r
summary(mtcars$mpg)
mean(mtcars$mpg); sd(mtcars$mpg); quantile(mtcars$mpg, probs = c(.25, .5, .75))
```

### 10.2 t-test (two groups)
```r
mt <- mtcars |>
  mutate(am = factor(am, labels = c("auto","manual")))

t.test(mpg ~ am, data = mt, var.equal = FALSE)  # Welch t-test
```

### 10.3 Linear Regression
```r
fit <- lm(mpg ~ wt + hp, data = mtcars)
summary(fit)
confint(fit)
predict(fit, newdata = data.frame(wt = 3, hp = 110), interval = "prediction")
```

---

## 11. Debugging, Documentation, and Help
- **Documentation**: `?mean`, `help("mean")`, `example(lm)`, `vignette("dplyr")`  
- **Error Trace**: use `traceback()` after error; `debugonce(fun)`; insert `browser()` in code.  
- **Messages and Warnings**: `message("info")`, `warning("careful")`, `stop("error")`.  

---

## 12. Reproducible Research and Project Structure
### 12.1 Random Seed and Session Info
```r
set.seed(123)
sessionInfo()
```

### 12.2 Recommended Project Layout
```
your-project/
├─ data/        # raw/external data (read-only)
├─ output/      # results (plots, tables, models)
├─ R/           # R scripts (functions, workflows)
├─ renv/        # (optional) package snapshots
└─ README.md    # project description and steps
```

### 12.3 Lock Package Versions (optional)
```r
install.packages("renv")
renv::init()         # create environment & snapshot
renv::snapshot()     # record versions
renv::restore()      # reproduce on another machine
```

---

## 13. Common Pitfalls and Tips
- **Strings vs Factors**: modern R/tibble does not auto-convert strings to factors; check types explicitly.  
- **Paths and Encoding**: use `here::here()` and UTF-8 for cross-platform.  
- **`==` with Floats**: avoid strict equality on floats; use `all.equal()` or tolerances.  
- **NA Handling**: use `na.rm = TRUE` in summary functions.  
- **Performance**: prefer vectorization, `apply`/`purrr` instead of unnecessary loops; preallocate objects.  

---

## 14. Mini Exercises
1. Load `mtcars`, convert `am` into factor with levels `auto/manual`, and compute mean/median mpg by group.  
2. Add column `Sepal.Ratio = Sepal.Length / Sepal.Width` to `iris` and plot boxplots by `Species`.  
3. Write function `zscore(x)` returning `(x - mean(x)) / sd(x)`; apply to `mtcars$wt` and store in new column.  
4. Convert table from wide to long and compute value sum by id:  
   ```
   id  A  B  C
   1  10  5  3
   2  20  6  4
   ```

---

## Appendix A: Function Cheat Sheet
- **Basics**: `c()`, `seq()`, `rep()`, `length()`, `sort()`, `unique()`  
- **Check**: `class()`, `typeof()`, `is.na()`, `anyNA()`, `str()`  
- **Indexing**: `[ ]`, `[[ ]]`, `$`  
- **Summary**: `mean()`, `median()`, `sd()`, `var()`, `summary()`, `quantile()`  
- **Data Frames**: `data.frame()`, `as_tibble()`, `subset()`, `merge()`  
- **dplyr**: `select()`, `rename()`, `mutate()`, `filter()`, `arrange()`, `summarise()`, `group_by()`, `across()`, joins  
- **tidyr**: `pivot_longer()`, `pivot_wider()`, `separate()`, `unite()`  
- **ggplot2**: `ggplot()`, `geom_point()`, `geom_line()`, `geom_bar()`, `geom_boxplot()`, `facet_wrap()`, `labs()`, `theme_*()`, `ggsave()`  
- **I/O**: `read_csv()`, `write_csv()`, `read_excel()`, `write_xlsx()`, `saveRDS()`, `readRDS()`  
- **Statistics**: `t.test()`, `lm()`, `aov()`, `cor()`, `prcomp()`  
- **Help**: `?fun`, `help()`, `example()`, `vignette()`, `traceback()`  

---

**License**: This tutorial is free to use and modify (CC BY 4.0).  
