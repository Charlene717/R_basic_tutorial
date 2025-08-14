# R 基本教程（入門到實用）
作者：Charlene  
版本：v2.0（整合版）  
最後更新：2025-08-14

> 面向初學者與需要快速複習的研究者。以 **R + tidyverse** 為主，示例資料使用內建 `iris`、`mtcars` 與 CSV。可直接作為 Markdown（.md）或 R Markdown（.Rmd）/ Quarto（.qmd）骨架檔。  
> 本版在你提供的內容基礎上，**保留原教學大部分結構**並**補齊未涵蓋的部分**（字串/日期、tidy evaluation、Rmd/Quarto 範例、I/O 格式延伸、除錯與效能等），並統整為一致的風格與用法。

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
- [授權](#授權)

---

## 0. 安裝與環境
- 下載 R：CRAN（https://cran.r-project.org/），建議搭配 **RStudio Desktop** 或 **Quarto CLI**。
- 中文/跨平台建議：全專案統一 **UTF-8** 編碼，避免路徑含空白或全形字元。
- 推薦套件一次安裝：
```r
install.packages(c(
  "tidyverse",  # dplyr, ggplot2, readr, tidyr, tibble, purrr, stringr, forcats
  "readxl", "writexl",    # Excel 讀寫
  "janitor",              # 欄名清理/資料品質
  "lubridate",            # 日期時間
  "here",                 # 穩定相對路徑
  "arrow", "qs",          # feather/ parquet / 高速序列化
  "renv",                 # 環境凍結
  "rmarkdown", "knitr",   # R Markdown
  "quarto"                # 若使用 R 包裝的 Quarto 介面（可選）
))
```
- 建議用 **RStudio Project** 或 **Quarto Project** 管理工作目錄；避免在腳本中 `setwd()`。

---

## 1. 基本語法與求助
```r
# 指派與列印
x <- 3 * 7
x  # 21

# 取得說明
?mean                     # 或 help(mean)
help.search("linear model")
apropos("lm")             # 模糊搜尋物件

# 工作目錄（僅查看）
getwd()
# setwd("/path/to/project") # 不建議硬編，見 §11

# 管線（R 4.1+ 原生 |>；亦可使用 magrittr::%>%）
mtcars |> head(3) |> summary()
```
- **向量化思維**：大多數運算可直接對向量操作，效能優於明確的 for 迴圈。
- **物件檢視**：`class()`、`typeof()`、`str()`、`summary()`。

---

## 2. 物件與資料結構
```r
# 向量（atomic vector）
v <- c(1, 2, 3); typeof(v)   # "double"

# 因子（類別型）
f <- factor(c("A","B","A"), levels = c("A","B","C"))

# 矩陣/陣列
m <- matrix(1:6, nrow = 2)

# 清單（list）
lst <- list(num = 1:3, chr = c("a","b"), flag = TRUE)

# 資料框與 tibble
df <- data.frame(x = 1:3, y = c("a","b","c"))
library(tibble)
tb <- tibble(x = 1:3, y = c("a","b","c"))
```
- **tibble vs data.frame**：tibble 不自動轉因子、印出更友善；與 dplyr/ggplot2 無縫。

---

## 3. 索引與子集
```r
x <- c(10, 20, 30, 40)
x[2]            # 20（位置）
x[c(1,4)]       # 10, 40（多位置）
x[-1]           # 移除第一個
x[x > 15]       # 條件式

# 資料框
iris[1:3, c("Sepal.Length","Species")]
iris$Species

# list：[] 保留 list；[[ ]] 抽出元素
lst <- list(a = 1:3, b = letters[1:2])
lst["a"]        # list，含 a
lst[["a"]]      # 向量 1:3

# 重要小訣竅
mt <- mtcars
mt[ , "mpg", drop = FALSE]  # 保留資料框形狀
which(mt$mpg > 25)          # 回傳符合條件的索引
match(c("am","hp"), names(mt))  # 由名稱找位置
```
- **tibble 子集**：`tb[ , "col"]` 回傳 tibble；`tb[["col"]]` 回傳向量。

---

## 4. 流程控制
```r
# if / else
x <- 5
if (x > 3) "big" else "small"

# for / while
s <- 0
for (i in 1:5) s <- s + i

i <- 1
while (i <= 3) { print(i); i <- i + 1 }

# 向量化與 ifelse / case_when
x <- -2:2
ifelse(x >= 0, "nonneg", "neg")

dplyr::case_when(
  x < 0  ~ "neg",
  x == 0 ~ "zero",
  TRUE   ~ "pos"
)
```
- **Apply 家族與 purrr（概念）**：以函數映射取代明顯迴圈；詳見 §7.5。  
- **注意**：`if (cond)` 的 `cond` 應為長度 1 的邏輯值；向量情境請用 `ifelse()` / `case_when()`。

---

## 5. 自訂函式
```r
# 具名參數 + 預設值 + 明確回傳
zscore <- function(x, na.rm = TRUE) {
  stopifnot(is.numeric(x))
  if (na.rm) x <- x[!is.na(x)]
  (x - mean(x)) / sd(x)
}
zscore(c(1, 2, 3, NA))

# '...' 可轉傳其他參數
my_mean <- function(x, ...) mean(x, ...)

# 隱性回傳 & 靜默回傳
f1 <- function(x) x + 1          # 最後一行即回傳
f2 <- function(x) invisible(x)   # 回傳但不自動列印
```
- **簡單防呆**：`stopifnot()`、或 `rlang::abort()`（tidyverse 風格）。

---

## 6. 導入與輸出 I/O
```r
library(readr)
# CSV：指定 encoding、型別與 NA 字串
cars <- read_csv("data/cars.csv",
                 locale   = locale(encoding = "UTF-8"),
                 col_types = cols(.default = col_guess()),
                 na = c("", "NA", "null"))
write_csv(cars, "export/cars_out.csv")

# Excel
library(readxl); library(writexl)
xl  <- read_excel("data/demo.xlsx", sheet = 1)
write_xlsx(xl, "export/demo_out.xlsx")

# R 專用序列化
iris <- readRDS("export/iris.rds")
saveRDS(iris, "export/iris.rds")

# feather / parquet（跨語言、高速 I/O）
library(arrow)
write_feather(mtcars, "export/mtcars.feather")
write_parquet(mtcars, "export/mtcars.parquet")

# 大物件高速壓縮
library(qs)
qsave(mtcars, "export/mtcars.qs")
mt_qs <- qread("export/mtcars.qs")
```
- **Windows 中文 Excel**：若需「含 BOM」CSV，可用 `readr::write_excel_csv()`。
- **型別安全**：`col_types` 明確指定欄型，避免推斷錯誤。

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
  summarise(n = n(), sl_mean = mean(Sepal.Length), .groups = "drop") |>
  arrange(desc(sl_mean))
res
```

### 7.1 更多常用語法
```r
# 欄操作
df <- as_tibble(mtcars, rownames = "model")
df |>
  rename(weight_1000lb = wt) |>
  relocate(model, .before = 1) |>
  mutate(power_wt = hp / weight_1000lb,
         am = factor(am, labels = c("auto","manual"))) |>
  distinct(model, .keep_all = TRUE)

# 條件轉換
df |>
  mutate(mpg_band = case_when(
    mpg >= 30            ~ "high",
    mpg >= 20 & mpg < 30 ~ "mid",
    TRUE                 ~ "low"
  ))

# 分組計算 + across
df |>
  group_by(am) |>
  summarise(across(c(mpg, hp, qsec), list(mean = mean, sd = sd), .names = "{.col}_{.fn}"),
            .groups = "drop")

# 缺失值處理
tibble(x = c(1, NA, 3), y = c(10, 20, NA)) |>
  mutate(x = coalesce(x, 0),
         y = replace_na(y, 99))
```

### 7.2 連接（join）
```r
spec_code <- tibble(Species = unique(iris$Species), code = 1:3)
count_tbl <- iris |>
  as_tibble() |>
  count(Species, name = "n")

left_join(spec_code, count_tbl, by = "Species")
inner_join(spec_code, count_tbl, by = "Species")
full_join(spec_code,  count_tbl, by = "Species")
anti_join(spec_code,  count_tbl, by = "Species")  # 左表有右表沒有的鍵
```

### 7.3 窗口函數與排序
```r
df |>
  arrange(desc(mpg)) |>
  mutate(rank_mpg = row_number(desc(mpg)),
         lag_mpg  = lag(mpg),
         lead_mpg = lead(mpg))
```

### 7.4 tidy evaluation（基礎）
當欄名作為參數傳入：
```r
mean_by <- function(data, grp, var) {
  data |>
    group_by({{ grp }}) |>
    summarise(m = mean({{ var }}), .groups = "drop")
}
mean_by(iris, Species, Sepal.Length)
```

### 7.5 purrr：函數式映射
```r
library(purrr)

# 對清單/列做函數，回傳型別受控
list(1:3, 4:6) |> map_dbl(mean)

# 對資料框每欄套用（小心回傳型別）
iris |>
  as_tibble() |>
  map_if(is.numeric, mean) |> unlist()

# 安全映射（錯誤捕捉）
safe_log <- safely(log)
map(list(1, "a", 10), ~ safe_log(.x))
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
se <- separate(tibble(id = c("A-1","B-2")),
               col = id, into = c("grp","idx"), sep = "-")
unite(se, col = id2, grp, idx, sep = "_")
```
- **缺失列處理**：`drop_na()`；填補請用 `tidyr::replace_na()` 或 dplyr 的 `coalesce()`。

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

# 直方圖 / 密度圖
ggplot(mtcars, aes(mpg)) + geom_histogram(bins = 15)
ggplot(mtcars, aes(mpg)) + geom_density()

# 儲存圖片
ggsave("export/iris_facets.png", p, width = 6, height = 4, dpi = 300)
```

---

## 10. 字串與日期
```r
library(stringr); library(lubridate)

# 字串（regex by default）
str_detect("EGFR_mutant", "mutant")             # TRUE
str_replace_all("a,b;c", "[;,]", "|")           # "a|b|c"
str_extract_all("geneA:TP53; geneB:EGFR", "(?<=:)\\w+")

# 日期時間（自動解析器）
ymd("2025-08-14")
mdy_hm("08/14/2025 09:00")
with_tz(now(), "Asia/Taipei")

# 運算與取整
d <- ymd("2025-08-14")
d + days(7); floor_date(now(), "month"); ceiling_date(now(), "week")
```
- **時區**：運算請明確指定時區；跨國資料避免混用當地與 UTC。

---

## 11. 專案與路徑管理
- 建立 **RStudio Project / Quarto Project**（建議一個專案對應一個研究/報告）。
- 以 `here::here()` 產生穩定相對路徑；避免 `setwd()`。
```r
library(here)
# 先在專案根目錄建立 "data/iris.csv"
readr::write_csv(iris, here("data", "iris.csv"))
dat <- readr::read_csv(here("data", "iris.csv"))
```
- 檔案與資料夾建議：
```
your-project/
├─ data/        # 原始/外部資料（唯讀）
├─ export/      # 匯出（表格、圖檔、模型）
├─ R/           # 自訂函式與分析腳本
├─ reports/     # Rmd / qmd / html / pdf
├─ renv/        # 套件快照
└─ README.md    # 專案說明與重現步驟
```

---

## 12. 可重現性：seed、session、renv
```r
set.seed(1234)     # 固定隨機性
sessionInfo()      # 記錄環境

# 套件版本凍結（專案層級）
library(renv)
renv::init()       # 建立專案私有 library 與 renv.lock
renv::snapshot()   # 記錄套件版本
# 另一台機器復原
renv::restore()
```
- **建議**：在報告中附上 `sessionInfo()` 或 `sessioninfo::session_info()`。

---

## 13. R Markdown / Quarto 範例
**R Markdown .Rmd 最小骨架：**
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

**Quarto .qmd 最小骨架：**
```markdown
---
title: "My Analysis"
author: "Your Name"
format: html
execute:
  echo: true
  warning: false
  message: false
---

## Intro
Some text.

```{r}
library(tidyverse)
iris |> group_by(Species) |> summarise(across(everything(), mean))
```
```
- **編譯**：Rmd 用 *Knit* 或 `rmarkdown::render()`；Quarto 用 `quarto render`。  
- **參數化報告**（Rmd 範例）：
```yaml
---
title: "Param Report"
params:
  cutoff: 5
output: html_document
---
```
```r
# 使用參數
subset_iris <- subset(iris, Sepal.Length > params$cutoff)
```

---

## 14. 除錯與效能
```r
# 立即捕捉錯誤
f <- function(x) { stop("boom") }
try(f(1))            # 不中斷流程，回傳 try-error

# 呼叫堆疊
traceback()

# 逐步偵錯
debugonce(mean); mean(1:3)
browser()            # 在程式中設中斷點

# 出錯即進入互動偵錯（暫時）
options(error = recover)
```
**效能量測與剖析：**
```r
system.time({ tmp <- lapply(1:1e4, sqrt) })   # 粗略時間
# install.packages("bench"); install.packages("profvis")
bench::mark(sqrt(1:1e6), (1:1e6) ^ 0.5)       # 比較兩種作法
profvis::profvis({ for (i in 1:1e5) sqrt(i) })# 互動式剖析
```
- **平行化**：`parallel::mclapply()`（Unix/mac），Windows 可用 `parallel::parLapply()` 或 `future.apply`。

---

## 15. 常見坑
- `NA` 比較：用 `is.na(x)`，不要 `x == NA`；彙總加 `na.rm = TRUE`。
- 因子與字串：確保型別正確；需要排序時 `factor(x, levels = ...)` 或 `forcats::fct_relevel()`。
- 路徑硬編 `setwd()`：改用專案與 `here::here()`。
- 浮點數比較：`all.equal(a, b)` 或 `abs(a-b) < 1e-8`。
- 迴圈效能：優先向量化或 `lapply()/purrr::map_*()`。
- 分組運算：`summarise()` 回傳 1 列/組；跨組依賴請用 `ungroup()` 或 `group_by()` 正確設置。
- 日期時區：混用當地/UTC 會出現 ±8 小時位移；請明確指定。

---

## 16. 速查表
```r
# 讀寫
readr::read_csv(); readr::write_csv()
readxl::read_excel(); writexl::write_xlsx()
saveRDS(); readRDS()
arrow::write_feather(); arrow::write_parquet(); qs::qsave(); qs::qread()

# dplyr
select(); filter(); mutate(); summarise(); group_by(); arrange(); distinct()
across(); case_when(); relocate(); count(); coalesce(); lag(); lead()
left_join(); right_join(); inner_join(); full_join(); anti_join()

# tidyr
pivot_longer(); pivot_wider(); separate(); unite(); drop_na(); replace_na()

# ggplot2
ggplot(); geom_point(); geom_line(); geom_boxplot(); geom_histogram(); geom_density()
facet_wrap(); labs(); theme_minimal(); ggsave()

# 字串與日期
stringr::str_detect(); stringr::str_replace_all(); stringr::str_extract_all()
lubridate::ymd(); lubridate::mdy_hm(); lubridate::with_tz()

# 其他
here::here(); janitor::clean_names(); purrr::map_*()
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
readr::write_csv(sum_tbl, here("export","summary.csv"))
ggsave(here("export","scatter.png"), plot = plt, width = 6, height = 4, dpi = 300)
```

---

## 授權
本教學可自由使用與修改（CC BY 4.0）。
