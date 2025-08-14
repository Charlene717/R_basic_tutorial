# R 基本教程
作者：Charlene  
版本：v1.0  
最後更新：2025-08-14

> 本教程面向剛開始使用 R 的讀者，涵蓋安裝、語法、資料結構、資料載入與清理、視覺化、基本統計、可重現研究等。所有示例可在純 R 或 RStudio 中直接執行。

---

## 目錄
- [1. 安裝與環境建議](#1-安裝與環境建議)
- [2. R 的最基本概念](#2-r-的最基本概念)
- [3. 基本語法與物件](#3-基本語法與物件)
- [4. 資料結構](#4-資料結構)
- [5. 讀寫資料（I/O）](#5-讀寫資料io)
- [6. 資料處理：base R 與 tidyverse](#6-資料處理base-r-與-tidyverse)
- [7. 視覺化：base plot 與 ggplot2](#7-視覺化base-plot-與-ggplot2)
- [8. 控制流程與自訂函數](#8-控制流程與自訂函數)
- [9. Apply 家族與 purrr](#9-apply-家族與-purrr)
- [10. 基本統計分析](#10-基本統計分析)
- [11. 除錯、說明文件與求助](#11-除錯說明文件與求助)
- [12. 可重現研究與專案結構](#12-可重現研究與專案結構)
- [13. 常見陷阱與小技巧](#13-常見陷阱與小技巧)
- [14. 迷你練習題](#14-迷你練習題)
- [附錄A：常用函數速查表](#附錄a常用函數速查表)

---

## 1. 安裝與環境建議
- **安裝 R**：建議使用最新穩定版（≥ 4.3）。
- **RStudio（可選）**：提供更友善的整合開發環境（IDE）。
- **套件管理**：
  - 安裝套件：
    ```r
    install.packages("tidyverse")   # dplyr、ggplot2、readr、tidyr 等
    install.packages("readxl")      # 讀取 Excel
    install.packages("writexl")     # 寫出 Excel
    install.packages("here")        # 管理專案路徑
    install.packages("janitor")     # 清理欄位名稱/資料品質檢查
    ```
  - 載入套件：
    ```r
    library(tidyverse)
    library(readxl); library(writexl)
    library(here);    library(janitor)
    ```
- **專案化**：建立 R 專案（RStudio: `File > New Project`），用 `here::here()` 管理相對路徑，避免 `setwd()`。

---

## 2. R 的最基本概念
- **R 是一種向量化語言**：許多運算會自動在向量層級進行。
- **物件導向但輕量**：常見類別（class）包括 `numeric`、`character`、`logical`、`factor`、`list`、`data.frame`。
- **指派（assignment）**：
  ```r
  x <- 1 + 2    # 慣用
  y = 3 + 4     # 亦可
  x; y
  ```
- **管線（pipe）**：R 4.1+ 提供 base pipe `|>`；tidyverse 也常用 `%>%`。
  ```r
  1:5 |> sum()
  # 等同於
  sum(1:5)
  ```

---

## 3. 基本語法與物件
### 3.1 基本型別
```r
a <- 1L           # integer
b <- 3.14         # numeric/double
c <- "hello"      # character
d <- TRUE         # logical
e <- as.factor(c("A","B","A"))  # factor
typeof(a); class(a)
```

### 3.2 運算與比較
```r
x <- c(1, 2, 3)
y <- c(10, 20, 30)
x + y          # 向量加總
x * 2          # 標量乘法
x^2            # 逐元素平方
x > 1          # 比較運算，回傳 logical 向量
```

### 3.3 缺失值 NA
```r
z <- c(1, NA, 3)
mean(z)              # NA
mean(z, na.rm = TRUE)  # 2
is.na(z)             # 檢查
```

### 3.4 文字處理（基礎）
```r
paste("R", "rocks")       # "R rocks"
paste0("id_", 1:3)        # "id_1" "id_2" "id_3"
nchar("資料科學")           # 字元數
toupper("abc"); tolower("AbC")
```

---

## 4. 資料結構
### 4.1 向量（vector）
```r
v <- c(3,1,4,1,5)
v[1]          # 第一個元素
v[2:4]        # 第2到第4
v[v > 2]      # 條件式篩選
sort(v); unique(v); length(v)
```

### 4.2 矩陣與陣列（matrix/array）
```r
m <- matrix(1:6, nrow = 2, byrow = TRUE)
m[1, 2]       # 第1列第2行
dim(m)        # 維度
```

### 4.3 清單（list）
```r
lst <- list(num = 1:3, ch = c("a","b"), flag = TRUE)
lst$num
lst[["ch"]]
```

### 4.4 資料框（data.frame）與 tibble
```r
df <- data.frame(id = 1:3, grp = c("A","B","A"), val = c(10, 20, 30))
df$val
df[ df$grp == "A", ]
tibble_df <- tibble::tibble(id = 1:3, grp = c("A","B","A"), val = c(10,20,30))
tibble_df
```

### 4.5 因子（factor）
```r
f <- factor(c("low","high","medium"), levels = c("low","medium","high"), ordered = TRUE)
f
```

---

## 5. 讀寫資料（I/O）
### 5.1 CSV / TSV
```r
library(readr)
dat <- read_csv("data/input.csv")       # 自動推斷欄位型別
write_csv(dat, "output/result.csv")
```

### 5.2 Excel
```r
library(readxl); library(writexl)
excel_dat <- read_excel("data/input.xlsx", sheet = 1)
write_xlsx(excel_dat, "output/result.xlsx")
```

### 5.3 RDS（R 內建序列化格式）
```r
saveRDS(dat, "output/dat.rds")
dat2 <- readRDS("output/dat.rds")
```

### 5.4 相對路徑（建議）
```r
library(here)
read_csv(here("data", "input.csv"))
```

> 建議以「專案目錄」為根目錄，資料放在 `data/`、腳本在 `R/`、輸出在 `output/`。

---

## 6. 資料處理：base R 與 tidyverse
以下以內建資料集 `mtcars` 與 `iris` 範例。

### 6.1 base R 常見操作
```r
head(mtcars, 3)
summary(mtcars)
mtcars$am <- factor(mtcars$am, labels = c("auto","manual"))
mtcars[mtcars$mpg > 25, c("mpg","hp","wt")]
```

### 6.2 dplyr 基本動詞
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

### 6.3 新增／轉換欄位
```r
iris |>
  as_tibble() |>
  mutate(Petal.Ratio = Petal.Length / Petal.Width,
         Species = as.character(Species)) |>
  slice_head(n = 5)
```

### 6.4 寬長表轉換（tidyr）
```r
library(tidyr)

wide <- tibble(id = 1:3, A = c(10,20,30), B = c(5,6,7))
long <- wide |>
  pivot_longer(cols = A:B, names_to = "key", values_to = "value")

restored <- long |>
  pivot_wider(names_from = key, values_from = value)
```

### 6.5 合併（join）
```r
A <- tibble(id = 1:3, valA = c(10,20,30))
B <- tibble(id = c(2,3,4), valB = c(200,300,400))

A |> left_join(B, by = "id")
A |> inner_join(B, by = "id")
A |> full_join(B,  by = "id")
```

### 6.6 清理欄位名稱（janitor）
```r
library(janitor)
dirty <- tibble("First Name" = c("Amy","Ben"), "Score(%)" = c(90, 85))
clean_names(dirty)   # "first_name", "score"
```

---

## 7. 視覺化：base plot 與 ggplot2
### 7.1 base R 快速繪圖
```r
plot(mtcars$wt, mtcars$mpg,
     main = "MPG vs Weight", xlab = "Weight (1000 lbs)", ylab = "MPG")
abline(lm(mpg ~ wt, data = mtcars), lwd = 2)
```

### 7.2 ggplot2 基礎語法
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

### 7.3 分面與儲存圖片
```r
p <- ggplot(iris, aes(Sepal.Length, Sepal.Width)) +
  geom_point(alpha = 0.7) +
  facet_wrap(~ Species) +
  labs(title = "Iris by Species")
p

ggsave("output/iris_facets.png", p, width = 6, height = 4, dpi = 300)
```

---

## 8. 控制流程與自訂函數
### 8.1 條件、迴圈
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

### 8.2 自訂函數與預設參數
```r
add_and_scale <- function(x, y = 0, scale = 1) {
  stopifnot(is.numeric(x), is.numeric(y), is.numeric(scale))
  (x + y) * scale
}

add_and_scale(1:3, y = 2, scale = 0.5)
```

### 8.3 作用域與回傳
```r
f <- function(x) {
  y <- x + 1      # 函數內部變數
  return(y * 2)   # 顯式回傳
}
f(10)
```

---

## 9. Apply 家族與 purrr
### 9.1 base R 的 apply 家族
```r
m <- matrix(1:9, nrow = 3)
apply(m, 1, sum)    # 每一列相加
apply(m, 2, mean)   # 每一欄平均

l <- list(a = 1:3, b = 4:6)
lapply(l, mean)     # 回傳 list
sapply(l, mean)     # 盡量簡化回傳
```

### 9.2 purrr（tidyverse 函數式工具）
```r
library(purrr)

list(1:3, 4:6) |>
  map_dbl(mean)       # 以 double 回傳

# 對資料框的每欄做運算
iris |>
  as_tibble() |>
  map_if(is.numeric, mean) |>    # 篩選 numeric 欄
  unlist()
```

---

## 10. 基本統計分析
### 10.1 描述統計
```r
summary(mtcars$mpg)
mean(mtcars$mpg); sd(mtcars$mpg); quantile(mtcars$mpg, probs = c(.25, .5, .75))
```

### 10.2 t 檢定（兩組比較）
```r
mt <- mtcars |>
  mutate(am = factor(am, labels = c("auto","manual")))

t.test(mpg ~ am, data = mt, var.equal = FALSE)  # Welch t-test
```

### 10.3 線性回歸
```r
fit <- lm(mpg ~ wt + hp, data = mtcars)
summary(fit)
confint(fit)
predict(fit, newdata = data.frame(wt = 3, hp = 110), interval = "prediction")
```

---

## 11. 除錯、說明文件與求助
- **查看說明**：`?mean`、`help("mean")`、`example(lm)`、`vignette("dplyr")`
- **錯誤追蹤**：發生錯誤後用 `traceback()`；或在函數前呼叫 `debugonce(fun)`、於程式中插入 `browser()`。
- **訊息與警告**：`message("info")`、`warning("careful")`、`stop("error")`。

---

## 12. 可重現研究與專案結構
### 12.1 隨機種子、環境資訊
```r
set.seed(123)
sessionInfo()
```

### 12.2 推薦的專案目錄
```
your-project/
├─ data/        # 原始或外部資料（唯讀）
├─ output/      # 產出結果（圖表、表格、模型）
├─ R/           # R 腳本（函數、分析流程）
├─ renv/        # （可選）套件快照
└─ README.md    # 專案說明與重現步驟
```

### 12.3 套件版本鎖定（可選）
```r
install.packages("renv")
renv::init()         # 建立虛擬環境與快照
renv::snapshot()     # 記錄套件版本
renv::restore()      # 於另一台機器重建環境
```

---

## 13. 常見陷阱與小技巧
- **字串與因子**：現代 R/tibble 讀入字串不會自動轉為因子；仍請確認型別是否正確。
- **路徑與編碼**：跨平台建議使用 `here::here()` 與 UTF-8。
- **`==` 與浮點數**：避免對小數做嚴格相等比較，改用 `all.equal()` 或容忍誤差。
- **NA 處理**：彙總函數常需 `na.rm = TRUE`。
- **速度**：向量化、多使用 `apply`/`purrr`，避免不必要的大迴圈；預先配置物件大小。

---

## 14. 迷你練習題
1. 讀入 `mtcars`，把 `am` 轉成 `auto/manual` 兩水準因子，計算每組 `mpg` 平均與中位數。
2. 用 `iris` 新增欄位 `Sepal.Ratio = Sepal.Length / Sepal.Width`，並畫出各 `Species` 的箱形圖（箱線圖）。
3. 建立函數 `zscore(x)` 回傳 `(x - mean(x)) / sd(x)`，用於 `mtcars$wt` 並將結果存成新欄位。
4. 將下表從寬轉長，再依 `id` 計算 `value` 總和：
   ```
   id  A  B  C
   1  10  5  3
   2  20  6  4
   ```

---

## 附錄A：常用函數速查表
- **基本**：`c()`、`seq()`、`rep()`、`length()`、`sort()`、`unique()`
- **檢查**：`class()`、`typeof()`、`is.na()`、`anyNA()`、`str()`
- **索引**：`[ ]`、`[[ ]]`、`$`
- **彙總**：`mean()`、`median()`、`sd()`、`var()`、`summary()`、`quantile()`
- **資料框**：`data.frame()`、`as_tibble()`、`subset()`、`merge()`
- **dplyr**：`select()`、`rename()`、`mutate()`、`filter()`、`arrange()`、`summarise()`、`group_by()`、`across()`、`joins`
- **tidyr**：`pivot_longer()`、`pivot_wider()`、`separate()`、`unite()`
- **ggplot2**：`ggplot()`、`geom_point()`、`geom_line()`、`geom_bar()`、`geom_boxplot()`、`facet_wrap()`、`labs()`、`theme_*()`、`ggsave()`
- **I/O**：`read_csv()`、`write_csv()`、`read_excel()`、`write_xlsx()`、`saveRDS()`、`readRDS()`
- **統計**：`t.test()`、`lm()`、`aov()`、`cor()`、`prcomp()`
- **求助**：`?fun`、`help()`、`example()`、`vignette()`、`traceback()`

---

**版權**：本教學可自由使用與修改（CC BY 4.0）。
