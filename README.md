# R 基本教程
作者：Charlene  
版本：v1.1  
最後更新：2025-08-14

> 面向初學者與需要快速複習的研究者。本教程以 **R + tidyverse** 為主，示例資料使用內建 `iris`、`mtcars` 與 CSV。可直接作為 Markdown（.md）或 R Markdown（.Rmd）骨架檔。示例在 R（≥ 4.1）與 RStudio 中可直接執行。

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
- [12. 可重現性：seed、session、renv](#12-可重現性seed-session-renv)
- [13. R Markdown / Quarto 範例](#13-r-markdown--quarto-範例)
- [14. 除錯與效能](#14-除錯與效能)
- [15. 常見坑](#15-常見坑)
- [16. 速查表](#16-速查表)
- [附錄：範例小專案骨架](#附錄範例小專案骨架)

---

## 0. 安裝與環境
- 下載 R：<https://cran.r-project.org/>；建議搭配 **RStudio Desktop**。
- 建議版本：R ≥ 4.3（支援原生 `|>` 管線）。
- 套件安裝與載入（**安裝一次，使用時要載入 `library()`**）：
  ```r
  install.packages(c(
    "tidyverse","readr","readxl","writexl","janitor",
    "lubridate","stringr","here","arrow","qs",
    "renv","rmarkdown","knitr"
  ))
  # 需用時載入
  library(tidyverse)  # dplyr, ggplot2, readr, tidyr 等
  library(readxl); library(writexl)
  library(janitor); library(lubridate); library(stringr)
  library(here);    library(arrow);    library(qs)
  library(renv);    library(rmarkdown); library(knitr)
  ```
- 中文/跨平台建議：儲存檔案採 **UTF-8**，必要時 I/O 指定 `locale = locale(encoding = "UTF-8")`。

---

## 1. 基本語法與求助
```r
# 指派與列印
x <- 3 * 7
x  # 21

# 取得說明
?mean                 # 或 help(mean)
help.search("linear model")
apropos("lm")         # 模糊搜尋物件名稱

# 目前工作目錄（不建議硬編 setwd()，見 §11）
getwd()

# 管線（R 4.1+ 原生 |>；亦可用 magrittr::%>%）
mtcars |> head(3) |> summary()
```
- **向量化**：R 多數運算都能直接作用於向量（見 §4）。
- **註解**：`#` 後為註解；R 不支援區塊註解。

---

## 2. 物件與資料結構
```r
# 向量（atomic vector）
v <- c(1, 2, 3); typeof(v)   # "double"

# 因子（類別型）
f <- factor(c("A","B","A"), levels = c("A","B","C"), ordered = FALSE)

# 矩陣與陣列
m <- matrix(1:6, nrow = 2)

# 清單（list）
lst <- list(num = 1:3, chr = c("a","b"), flag = TRUE)

# 資料框與 tibble
df <- data.frame(x = 1:3, y = c("a","b","c"))
library(tibble)
tb <- tibble(x = 1:3, y = c("a","b","c"))
```
- **tibble vs data.frame**：tibble 讀入字串不會自動轉因子、列印更友善、錯誤訊息更清楚。  
- 型別與結構檢視：`class()`、`typeof()`、`str()`、`summary()`。

---

## 3. 索引與子集
```r
x <- c(10, 20, 30, 40)
x[2]          # 位置
x[c(1,4)]     # 多位置
x[-1]         # 移除第一個
x[x > 15]     # 條件式

# 資料框
iris[1:3, c("Sepal.Length","Species")]
iris$Species

# list：[] 保留 list；[[ ]] 抽出元素；$ 以名稱擷取
lst <- list(a = 1:3, b = letters[1:2])
lst["a"]      # list，含 a
lst[["a"]]    # 向量 1:3
lst$b         # "a" "b"
```
> 小技巧：tidyverse 常以 `filter()`、`select()` 進行子集化，較直觀（見 §7）。

---

## 4. 流程控制
```r
# if / else
x <- 5
if (x > 3) "big" else "small"

# for / while（向量化通常更快，見下）
s <- 0; for (i in 1:5) s <- s + i; s  # 15

# 向量化與 *apply
x <- 1:5
x2 <- x * 2   # 向量化
lapply(split(iris$Sepal.Length, iris$Species), mean)
sapply(split(iris$Sepal.Length, iris$Species), mean)  # 精簡回傳
```
- `apply(X, MARGIN, FUN)`：矩陣/陣列跨列（1）或跨欄（2）運算。  
- purrr 的 `map_*()` 家族提供型別穩定回傳（`map_dbl()` 等），進階見 `?purrr::map`。

---

## 5. 自訂函式
```r
# 具名參數 + 預設值 + 明確回傳 + 基本檢查
zscore <- function(x, na.rm = TRUE) {
  stopifnot(is.numeric(x))
  mu <- mean(x, na.rm = na.rm)
  sdv <- sd(x, na.rm = na.rm)
  (x - mu) / sdv
}

zscore(c(1, 2, 3, NA))
```

---

## 6. 導入與輸出 I/O
```r
library(readr)
# CSV（建議 UTF-8；Windows 可指定 locale）
cars <- read_csv("data/cars.csv", locale = locale(encoding = "UTF-8"))
write_csv(cars, "export/cars_out.csv")

# Excel
library(readxl); library(writexl)
xl  <- read_excel("data/demo.xlsx", sheet = 1)
write_xlsx(xl, "export/demo_out.xlsx")

# R 專用序列化（單一 R 物件）
iris <- readRDS("export/iris.rds")
saveRDS(iris, "export/iris.rds")

# 高速/跨語言（可選）
library(arrow); write_feather(mtcars, "export/mtcars.feather")

# 大物件壓縮（可選）
library(qs); qsave(mtcars, "export/mtcars.qs"); qread("export/mtcars.qs")

# 其他（選讀）
# jsonlite::fromJSON("data/file.json"); jsonlite::toJSON(obj, pretty = TRUE)
```
> 建議以「專案目錄」為根目錄：資料放在 `data/`、輸出在 `export/`，用 `here::here()` 管理路徑（§11）。

---

## 7. 資料處理（dplyr）
```r
library(dplyr)
# 基本動詞：select / filter / arrange / mutate / summarise / group_by
res <- iris |> 
  as_tibble() |> 
  select(Species, Sepal.Length:Petal.Width) |> 
  filter(Sepal.Length > 5) |> 
  mutate(SL_cm = Sepal.Length,
         ratio = Petal.Length / Sepal.Length) |> 
  group_by(Species) |> 
  summarise(n = n(),
            sl_mean = mean(Sepal.Length, na.rm = TRUE),
            .groups = "drop") |> 
  arrange(desc(sl_mean))
res
```
- 其他常用：`distinct()`、`rename()`、`relocate()`、`case_when()`、`across()`。
```r
# across + 重新命名；distinct 範例
iris |> 
  as_tibble() |>
  group_by(Species) |>
  summarise(across(starts_with("Sepal"),
                   list(mean = ~mean(.x, na.rm = TRUE),
                        sd   = ~sd(.x,   na.rm = TRUE)),
                   .names = "{.col}_{.fn}")) |>
  distinct()
```
- **連接（join）**
```r
left_join(
  tibble(Species = unique(iris$Species), code = 1:3),
  iris |> as_tibble() |> count(Species, name = "n"),
  by = "Species"
)
```
- **tidy evaluation（基礎）**：當欄名作為參數傳入
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

# 缺失值處理
drop_na(long)                # 移除含 NA 的列
replace_na(list(value = 0))  # 指定欄位補值
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
  labs(title = "Iris: Petal vs Sepal by Species",
       x = "Sepal Length", y = "Petal Length", color = "Species") +
  theme_minimal()
p

# 盒鬚圖 + 抖動
ggplot(iris, aes(Species, Sepal.Length)) +
  geom_boxplot() +
  geom_jitter(width = 0.2, alpha = 0.5)

# 儲存圖片
ggsave("export/iris_facets.png", p, width = 6, height = 4, dpi = 300)
```

---

## 10. 字串與日期
```r
library(stringr); library(lubridate)
# 字串
str_detect("EGFR_mutant", "mutant")           # TRUE
str_replace_all("a,b;c", "[;,]", "|")         # "a|b|c"
str_c("gene", 1:3, sep = "_")                 # "gene_1" "gene_2" "gene_3"

# 日期時間（自動剖析與時區）
ymd("2025-08-14")
mdy_hm("08/14/2025 09:00")
now() |> with_tz("Asia/Taipei")
```
> 正規表示式（regex）常用：`.` 任意字元、`^` 行首、`$` 行尾、`[]` 字元集合、`()` 群組、`|` 或。

---

## 11. 專案與路徑管理
- 使用 **RStudio Project** 或 **Quarto Project** 管理工作目錄。
- 以 `here::here()` 產生穩定相對路徑（避免 `setwd()`）：
  ```r
  library(here)
  # 先在專案根目錄建立 "data/iris.csv"
  readr::write_csv(iris, here("data", "iris.csv"))
  dat <- readr::read_csv(here("data", "iris.csv"))
  ```
- 建議目錄結構：
  ```
  your-project/
  ├─ data/        # 原始或外部資料（唯讀）
  ├─ export/      # 產出結果（圖表、表格、模型）
  ├─ R/           # 函式與分析腳本
  ├─ renv/        # （可選）套件快照
  └─ README.md    # 專案說明與重現步驟
  ```

---

## 12. 可重現性：seed、session、renv
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
**R Markdown（.Rmd）最小骨架：**
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
**Quarto（.qmd）** 類似，安裝 Quarto 後以 `quarto render` 編譯；RStudio 的 **Knit** 按鈕亦可。

---

## 14. 除錯與效能
```r
# 除錯
f <- function(x) { stop("boom") }
try(f(1))           # 捕捉錯誤（不中斷流程）
traceback()         # 呼叫堆疊
debugonce(mean); mean(1:3)  # 單次逐步
# 於程式中置入 browser() 進入互動除錯

# 基本效能評估
system.time({
  tmp <- lapply(1:1e3, sqrt)
})
```
> 需要更細緻的效能檢測可看 `bench` 或 `microbenchmark` 套件。

---

## 15. 常見坑
- **NA 比較**：使用 `is.na(x)`，不要用 `x == NA`。
- **因子與字串混用**：base 的 `data.frame()` 可設 `stringsAsFactors = FALSE`；tibble 預設不轉因子。
- **路徑硬編 `setwd()`**：改用專案與 `here()`。
- **回圈效能**：先考慮向量化或 `map_*()`/`lapply()`。
- **浮點數比較**：用 `all.equal(a, b)` 或 `abs(a-b) < 1e-8`。
- **回收（recycling）規則**：不同長度向量運算可能靜默回收，留意警告。

---

## 16. 速查表
```r
# 讀寫
readr::read_csv(); readr::write_csv()
readxl::read_excel(); writexl::write_xlsx()
saveRDS(); readRDS()
arrow::write_feather(); arrow::read_feather()

# dplyr
select(); filter(); mutate(); summarise(); group_by(); arrange(); distinct()
across(); case_when(); relocate()
left_join(); right_join(); inner_join(); full_join()

# tidyr
pivot_longer(); pivot_wider(); separate(); unite(); drop_na(); replace_na()

# ggplot2
ggplot(); geom_point(); geom_line(); geom_boxplot(); geom_histogram();
facet_wrap(); labs(); theme_minimal(); ggsave()

# 其他
stringr::str_detect(); stringr::str_replace_all()
lubridate::ymd(); lubridate::mdy_hm(); lubridate::now()
here::here(); janitor::clean_names()
```
---

## 附錄：範例小專案骨架
```r
# 專案啟動 ---------------------------------------------------------------
library(here); library(readr); library(dplyr); library(ggplot2); library(janitor)
readr::write_csv(iris, here("data","iris.csv"))

# 資料讀取 ---------------------------------------------------------------
dat <- read_csv(here("data","iris.csv"))

# 清理與特徵 -------------------------------------------------------------
dat2 <- dat |>
  janitor::clean_names() |>
  mutate(petal_ratio = petal_length / petal_width)

# 彙整 ---------------------------------------------------------------
sum_tbl <- dat2 |>
  group_by(species) |>
  summarise(across(starts_with("sepal_"), mean), .groups = "drop")

# 視覺化 ---------------------------------------------------------------
plt <- ggplot(dat2, aes(sepal_length, petal_length, color = species)) +
  geom_point(alpha = 0.7) + theme_minimal()
print(plt)

# 匯出 ---------------------------------------------------------------
write_csv(sum_tbl, here("export","summary.csv"))
ggsave(here("export","scatter.png"), plot = plt, width = 6, height = 4, dpi = 300)
```

---

**版權**：本教學可自由使用與修改（CC BY 4.0）。
