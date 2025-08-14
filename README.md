# R 基本教程（入門到實用）

> 面向初學者與需要快速複習的研究者。本教程以 **R + tidyverse** 為主，示例資料使用內建 `iris`、`mtcars` 與 CSV。可直接作為 Markdown（.md）或 R Markdown（.Rmd）骨架檔。

---

## 目錄
- [0. 安裝與環境](#0-安裝與環境)
- [1. 基本語法與求助](#1-基本語法與求助)
- [2. 物件與資料結構](#2-物件與資料結構)
- [3. 索引與子集](#3-索引與子集)
- [4. 流程控制](#4-流程控制)
- [5. 自訂函式](#5-自訂函式)
- [6. 導入與輸出 I/O](#6-導入與輸出-io)
- [7. 資料處理（dplyr）](#7-資料處理dplyr)
- [8. 欄列轉換（tidyr）](#8-欄列轉換tidyr)
- [9. 視覺化（ggplot2）](#9-視覺化ggplot2)
- [10. 字串與日期](#10-字串與日期)
- [11. 專案與路徑管理](#11-專案與路徑管理)
- [12. 可重現性：seed、session、renv](#12-可重現性seed-sessions-renv)
- [13. R Markdown / Quarto 範例](#13-r-markdown--quarto-範例)
- [14. 除錯與效能](#14-除錯與效能)
- [15. 常見坑](#15-常見坑)
- [16. 速查表](#16-速查表)

---

## 0. 安裝與環境
- 下載 R：[CRAN](https://cran.r-project.org/)；建議搭配 RStudio Desktop。
- 套件安裝：
```r
install.packages(c("tidyverse", "readxl", "writexl", "janitor", "lubridate", 
                   "stringr", "here", "arrow", "qs", "renv", "rmarkdown", "knitr"))
```
- 中文環境注意：讀寫檔時建議指定 `encoding = "UTF-8"` 或確保檔案為 UTF-8。

---

## 1. 基本語法與求助
```r
# 指派與列印
x <- 3 * 7
x  # 21

# 取得說明
?mean          # 或 help(mean)
help.search("linear model")
apropos("lm") # 模糊搜尋物件

# 工作目錄
getwd()
# setwd("/path/to/project")  # 不建議硬編；改用 here::here()，見 §11

# 管線（R 4.1+ 原生 |>）
mtcars |> head(3) |> summary()
```

---

## 2. 物件與資料結構
```r
# 向量（atomic vector）
v <- c(1, 2, 3); typeof(v)   # "double"

# 因子（類別型）
f <- factor(c("A","B","A"), levels = c("A","B","C"))

# 矩陣與陣列
m <- matrix(1:6, nrow = 2)

# 清單（list）
lst <- list(num = 1:3, chr = c("a","b"), flag = TRUE)

# 資料框（data.frame）與 tibble（tidyverse）
df <- data.frame(x = 1:3, y = c("a","b","c"))
library(tibble)
tb <- tibble(x = 1:3, y = c("a","b","c"))
```
- **tibble vs data.frame**：tibble 不會自動轉因子、不會部分列印時轉型，互動更友善。

---

## 3. 索引與子集
```r
x <- c(10, 20, 30, 40)
x[2]          # 20（位置）
x[c(1,4)]     # 10, 40（多位置）
x[-1]         # 移除第一個
x[x > 15]     # 條件式

# 資料框
iris[1:3, c("Sepal.Length","Species")]
iris$Species

# list：[] 保留 list；[[ ]] 抽出元素
lst <- list(a = 1:3, b = letters[1:2])
lst["a"]      # list，含 a
lst[["a"]]    # 向量 1:3
```

---

## 4. 流程控制
```r
# if / else
x <- 5
if (x > 3) {
  "big"
} else {
  "small"
}

# for / while
s <- 0
for (i in 1:5) s <- s + i
s  # 15

# 向量化與 *apply
x <- 1:5
x2 <- x * 2               # 向量化更快
lapply(split(iris$Sepal.Length, iris$Species), mean)
```

---

## 5. 自訂函式
```r
# 具名參數 + 預設值 + 明確回傳
zscore <- function(x, na.rm = TRUE) {
  if (na.rm) x <- x[!is.na(x)]
  (x - mean(x)) / sd(x)
}

zscore(c(1, 2, 3, NA))
```

---

## 6. 導入與輸出 I/O
```r
library(readr)
# CSV
cars <- read_csv("data/cars.csv", locale = locale(encoding = "UTF-8"))
write_csv(cars, "export/cars_out.csv")

# Excel
library(readxl); library(writexl)
xl  <- read_excel("data/demo.xlsx", sheet = 1)
write_xlsx(xl, "export/demo_out.xlsx")

# R 專用序列化
saveRDS(iris, "export/iris.rds")
iris2 <- readRDS("export/iris.rds")

# 高速/跨語言（可選）
library(arrow); write_feather(mtcars, "export/mtcars.feather")

# 大物件壓縮（可選）
library(qs); qsave(mtcars, "export/mtcars.qs"); qread("export/mtcars.qs")
```

---

## 7. 資料處理（dplyr）
```r
library(dplyr)
# 基本動詞：select / filter / arrange / mutate / summarise / group_by
res <- iris |> 
  as_tibble() |> 
  select(Species, Sepal.Length:Petal.Width) |> 
  filter(Sepal.Length > 5) |> 
  mutate(SL_cm = Sepal.Length, ratio = Petal.Length / Sepal.Length) |> 
  group_by(Species) |> 
  summarise(n = n(), sl_mean = mean(Sepal.Length), .groups = "drop") |> 
  arrange(desc(sl_mean))
res

# 連接（join）
left_join(tibble(Species = unique(iris$Species), code = 1:3), 
          summarise(group_by(iris |> as_tibble(), Species), n = n()),
          by = "Species")
```

### tidy evaluation（基礎）
當欄名作為參數傳入：
```r
mean_by <- function(data, grp, var) {
  data |> group_by({{ grp }}) |> summarise(m = mean({{ var }}), .groups = "drop")
}
mean_by(iris, Species, Sepal.Length)
```

---

## 8. 欄列轉換（tidyr）
```r
library(tidyr)
# 寬轉長
long <- pivot_longer(iris |> as_tibble(),
                     cols = Sepal.Length:Petal.Width,
                     names_to = "feature", values_to = "value")
# 長轉寬
wide <- pivot_wider(long, names_from = feature, values_from = value)

# 分割 / 合併
se <- separate(tibble(id = c("A-1","B-2")), col = id, into = c("grp","idx"), sep = "-")
unite(se, col = id2, grp, idx, sep = "_")
```

---

## 9. 視覺化（ggplot2）
```r
library(ggplot2)
# 散佈圖 + 擬合線 + 分面
p <- ggplot(iris, aes(Sepal.Length, Petal.Length, color = Species)) +
  geom_point(alpha = 0.7) +
  geom_smooth(method = "lm", se = FALSE) +
  facet_wrap(~ Species) +
  theme_minimal()
print(p)

# 盒鬚圖
ggplot(iris, aes(Species, Sepal.Length)) +
  geom_boxplot() +
  geom_jitter(width = 0.2, alpha = 0.5)
```

---

## 10. 字串與日期
```r
library(stringr); library(lubridate)
# 字串
str_detect("EGFR_mutant", "mutant")     # TRUE
str_replace_all("a,b;c", "[;,]", "|")    # "a|b|c"

# 日期時間
ymd("2025-08-14")
mdy_hm("08/14/2025 09:00")
now() |> with_tz("Asia/Taipei")
```

---

## 11. 專案與路徑管理
- 使用 **RStudio Project** 或 **Quarto Project** 管理工作目錄。
- 以 `here::here()` 產生穩定相對路徑：
```r
library(here)
# 先在專案根目錄建立 "data/iris.csv"
readr::write_csv(iris, here("data", "iris.csv"))
dat <- readr::read_csv(here("data", "iris.csv"))
```

---

## 12. 可重現性：seed, sessions, renv
```r
set.seed(1234)       # 固定隨機性
sessionInfo()        # 紀錄環境

# 專案層級套件凍結（第一次）
library(renv)
renv::init()         # 建立專案私有 library
renv::snapshot()     # 鎖定版本到 renv.lock
# 另一台機器復原
renv::restore()
```

---

## 13. R Markdown / Quarto 範例
**R Markdown .Rmd** 最小骨架：
```markdown
---
title: "My Analysis"
author: "Your Name"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE, warning = FALSE, message = FALSE)
```

# Intro
Some text.

```{r}
library(tidyverse)
iris |> group_by(Species) |> summarise(across(everything(), mean))
```
```

**Quarto .qmd** 幾乎相同，只需安裝 Quarto 並以 `quarto render` 編譯。

---

## 14. 除錯與效能
```r
# 除錯
f <- function(x) { stop("boom") }
try(f(1))            # 捕捉錯誤
traceback()          # 呼叫堆疊

# 逐步
debugonce(mean); mean(1:3)
# 或使用 browser() 置入中斷點

# 粗略效能
system.time({
  tmp <- lapply(1:1e3, sqrt)
})
```

---

## 15. 常見坑
- `NA` 比較：使用 `is.na(x)`，不要用 `x == NA`。
- 因子與字串混用：`stringsAsFactors = FALSE`（base），tibble 預設已是字串。
- 路徑硬編 `setwd()`：改用專案與 `here()`。
- 回圈效能差：先考慮向量化或 `map_*()`/`lapply()`。
- 浮點數比較：用 `all.equal(a, b)` 或 `abs(a-b) < 1e-8`。

---

## 16. 速查表
```r
# 讀寫
readr::read_csv(); readr::write_csv()
readxl::read_excel(); writexl::write_xlsx()
saveRDS(); readRDS()

# dplyr
select(); filter(); mutate(); summarise(); group_by(); arrange(); distinct()
across(); case_when(); relocate(); join 家族：left_join/right_join/inner_join/full_join

# tidyr
pivot_longer(); pivot_wider(); separate(); unite(); drop_na()

# ggplot2
ggplot(); geom_point(); geom_line(); geom_boxplot(); geom_histogram();
facet_wrap(); labs(); theme_minimal()

# 其他
stringr::str_detect(); lubridate::ymd(); here::here()
```

---

### 範例小專案骨架（可直接複製）
```r
# 專案啟動 ---------------------------------------------------------------
library(here); library(readr); library(dplyr); library(ggplot2)
readr::write_csv(iris, here("data","iris.csv"))

# 資料讀取 ---------------------------------------------------------------
dat <- read_csv(here("data","iris.csv"))

# 清理與特徵 -------------------------------------------------------------
dat2 <- dat |> 
  janitor::clean_names() |> 
  mutate(petal_ratio = petal_length / petal_width)

# 彙整 ---------------------------------------------------------------
sum_tbl <- dat2 |> group_by(species) |> summarise(across(starts_with("sepal_"), mean))

# 視覺化 ---------------------------------------------------------------
plt <- ggplot(dat2, aes(sepal_length, petal_length, color = species)) +
  geom_point(alpha = 0.7) + theme_minimal()
print(plt)

# 匯出 ---------------------------------------------------------------
write_csv(sum_tbl, here("export","summary.csv"))
ggsave(here("export","scatter.png"), plot = plt, width = 6, height = 4, dpi = 300)
```
---

> ✅ 建議：將本檔另存為 `R_basic_tutorial.md` 或改為 `.Rmd`，即可直接 Knit 成報告。若需我幫你客製化（加上你的資料與領域案例），請告訴我章節或需求。
