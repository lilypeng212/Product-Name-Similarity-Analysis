# Product-Name-Similarity-Analysis

---
"品名相似度"
Created Date: "2025-03-18"
---

```{r setup, include=FALSE}
library(dplyr)
library(stringdist)
library(readxl)
library(openxlsx)
library(fuzzyjoin) 
library(data.table)
library(readr)


上架商品_1P <- read_excel("C:/Users/Lily Peng/Downloads/2024Q3女裝有售商品.xlsx")


上架新品_3P <- read_excel("C:/Users/Lily Peng/OneDrive - McKinsey & Company/3P女裝商品.xlsx")


df_上架商品_1P <- data.frame(上架商品_1P)

df_上架新品_3P <- data.frame(上架新品_3P)

# df_上架商品_1P <- df_上架商品_1P %>%
#   mutate(
#     source = "1P",
#     商品代碼 = as.character(goods_code),
#     商品名稱 = goods_name
#   )
#   
# 
# df_上架商品_3P <- df_上架商品_3P %>%
#   mutate(source = "3P") %>% 
#   rename(商品代碼 = goods_code,
#          商品名稱 = goods_name)
# 
# df_上架商品_1P <- bind_rows(df_上架商品_1P, df_上架商品_3P)


df_上架商品_1P$商品名稱 <- as.character(df_上架商品_1P$商品名稱)
df_上架新品_3P$商品名稱 <- as.character(df_上架新品_3P$商品名稱)

```

amatch
```{r}
start_time <- Sys.time()
# 1. 過濾掉同品名 + 同品號的商品（保留同品名不同品號）
df_全站_no_3P <- df_上架商品_1P %>%
  anti_join(
    df_上架新品_3P %>% select(商品名稱, 商品代碼),
    by = c("商品名稱", "商品代碼")
  )

# 2. 進行 Jaccard 比對
best_match_index <- amatch(
  df_上架新品_3P$商品名稱,
  df_全站_no_3P$商品名稱,
  method = "jaccard",
  maxDist = 0.5
)

# 3. 過濾有效比對
valid_matches <- !is.na(best_match_index)

# 4. 建立比對結果資料框
df_best_match_2 <- df_上架新品_3P[valid_matches, ] %>%
  mutate(
    商品名稱_比對結果 = df_全站_no_3P$商品名稱[best_match_index[valid_matches]],
    相似度 = 1 - stringdist(
      商品名稱,
      商品名稱_比對結果,
      method = "jaccard"
    )
  )

end_time <- Sys.time()

write.xlsx(df_best_match_2, "商品相似度分析_3P女裝商品.xlsx", overwrite = TRUE)
```


計算相似商品業績
```{r}
# # 進行 left_join
# df_上架新品_3P_with_similarity <- df_上架新品_3P_with_similarity %>%
#   left_join(df_上架商品_1P, by = c("相似度高的商品" = "商品名稱"), relationship = "one-to-many") 
# 
# 
# df_上架新品_3P_with_similarity <- df_上架新品_3P_with_similarity %>%
#   rename(
#     商品代碼_1P = 商品代碼.y,  
#     商品名稱_1P = 相似度高的商品 
#   )

# # ✅ **計算 ASP**
# df_上架新品_3P_with_similarity <- df_上架新品_3P_with_similarity %>%
#   mutate(
#     ASP_3P = ifelse(淨訂購數量.x > 0, 淨訂購金額.x / 淨訂購數量.x, NA),
#     ASP_1P = ifelse(淨訂購數量.y > 0, 淨訂購金額.y / 淨訂購數量.y, NA),
#     ASP_1P_3P_diff = ASP_1P - ASP_3P  # ✅ 計算 ASP 差異
#   )
```


```{r}
# **將 `df_上架新品_3P` 與 `df_best_match` 進行合併**
df_best_match_3 <- df_best_match_2%>%
  left_join(df_全站_no_3P, by = c("商品名稱_比對結果" = "商品名稱")) 

# # **重新命名欄位**
# df_上架新品_3P_with_similarity <- df_上架新品_3P_with_similarity %>%
#   rename(
#     商品代碼_1P = 商品代碼.y,  
#     商品名稱_1P = 商品名稱_1P 
#   )

# # ✅ **依照 MD 部門排序，再依相似度排序**
# df_best_match <- df_best_match %>%
#   arrange(MD部別名稱.x, desc(相似度))

# ✅ **儲存結果到 Excel**
write.xlsx(df_best_match_3, "商品相似度分析_3P女裝商品_NEW.xlsx", overwrite = TRUE)



```
############faster version

---
title: "Untitled"
output: html_document
date: "2025-05-22"
---

```{r setup, include=FALSE}
library(dplyr)
library(stringdist)
library(readxl)
library(openxlsx)
library(fuzzyjoin)
library(data.table)
library(readr)
上架商品_1P <- read_excel("C:/Users/Lily Peng/OneDrive - McKinsey & Company/(EXT)-Shared folder_with momo team - Documents/2_mo+/999. Team working folder/Lily/0301-12_1P有售商品.xlsx",sheet=1)

上架新品_3P <- read_excel("C:/Users/Lily Peng/OneDrive - McKinsey & Company/(EXT)-Shared folder_with momo team - Documents/5_Traffic/999. Working folder/Kevin/4_ad-hoc analysis/0301-12_3P無銷售商品.xlsx",sheet=3)

df_上架商品_1P <- data.frame(上架商品_1P)
df_上架新品_3P <- data.frame(上架新品_3P)
df_上架商品_1P$商品名稱 <- as.character(df_上架商品_1P$商品名稱)
df_上架新品_3P$商品名稱 <- as.character(df_上架新品_3P$商品名稱)
```

```{r}
library(dplyr)
library(stringdist)
library(future.apply)
library(progressr)
# ─────────────────────────────────────────────────────────────────
# 0. （可選）設定讀檔、輸出路徑，如果你已經在外面做過就不需要
# ─────────────────────────────────────────────────────────────────
# （省略：df_上架商品_全站 與 df_上架新品_3P 已預先讀入）
# ─────────────────────────────────────────────────────────────────
# 1. 開始計時
# ─────────────────────────────────────────────────────────────────
start_time <- Sys.time()
# ─────────────────────────────────────────────────────────────────
# 2. 過濾出要比對的資料
# ─────────────────────────────────────────────────────────────────
df_全站_no_3P <- df_上架商品_1P %>%
  anti_join(df_上架新品_3P %>% select(商品名稱, 商品代碼),
            by = c("商品名稱", "商品代碼"))
vec_new <- df_上架新品_3P$商品名稱
vec_all <- df_全站_no_3P$商品名稱
# ─────────────────────────────────────────────────────────────────
# 3. 啟用多核心 + 進度條
# ─────────────────────────────────────────────────────────────────
plan(multisession, workers = parallel::detectCores() - 1)
handlers("txtprogressbar")
# ─────────────────────────────────────────────────────────────────
# 4. 平行 Jaccard fuzzy matching（maxDist = 0.4 等價）
# ─────────────────────────────────────────────────────────────────
res <- with_progress({
  p <- progressor(along = vec_new)
  future_lapply(seq_along(vec_new), function(i) {
    p()  # 每處理一筆，進度條自動推進
    dists <- stringdist(vec_new[i], vec_all, method = "jaccard")
    min_dist <- min(dists, na.rm = TRUE)
    if (min_dist <= 0.3) {
      idx <- which.min(dists)
      data.frame(
        原索引           = i,
        商品名稱_3P       = vec_new[i],
        商品名稱_比對結果 = vec_all[idx],
        相似度            = 1 - min_dist,
        stringsAsFactors   = FALSE
      )
    } else {
      NULL
    }
  })
})
# 回復單核心執行策略
plan(sequential)
# ─────────────────────────────────────────────────────────────────
# 5. 合併比對結果回原始新品表
# ─────────────────────────────────────────────────────────────────
df_best_match <- do.call(rbind, res)
df_best_match <- df_上架新品_3P[df_best_match$原索引, ] %>%
  mutate(
    商品名稱_比對結果 = df_best_match$商品名稱_比對結果,
    相似度         = df_best_match$相似度
  )
# ─────────────────────────────────────────────────────────────────
# 6. 結束計時並顯示總耗時
# ─────────────────────────────────────────────────────────────────
end_time <- Sys.time()
cat("★ 總耗時：", round(as.numeric(difftime(end_time, start_time, units="mins")), 1), "分鐘\n")
write.xlsx(df_best_match, "商品相似度分析_無銷售1100_2.xlsx", overwrite = TRUE)
```


```{r}
df_best_match <- df_best_match%>%
  left_join(df_上架商品_1P, by = c("商品名稱_比對結果" = "商品名稱"))
# :white_check_mark: **儲存結果到 Excel**
write.xlsx(df_best_match, "商品相似度分析_Alisa_speedtest.xlsx", overwrite = TRUE)
print(":white_check_mark: 商品相似度分析已完成，結果儲存至 '商品相似度分析.xlsx'！")






############for layer1

Title: "品名重複度(部門、第一層比對)"
output: html_document
date: "2025-05-19"
---
```{r}
library(dplyr)
library(stringdist)
library(openxlsx)
library(fuzzyjoin)
library(data.table)
library(stringr)
library(future.apply)
library(progressr)
library(readxl)


有售商品_1P <- read_excel("C:/Users/Lily Peng/Downloads/6月有售商品_1P.xlsx")

有售商品_3P <- read_excel("C:/Users/Lily Peng/Downloads/6月有售商品_3P.xlsx")


df_有售商品_1P <- data.frame(有售商品_1P)
df_有售商品_3P <- data.frame(有售商品_3P) 


df_有售商品_1P$商品名稱 <- as.character(df_有售商品_1P$商品名稱)
df_有售商品_3P$商品名稱 <- as.character(df_有售商品_3P$商品名稱)

df_有售商品_1P$第一層 <- str_trim(df_有售商品_1P$第一層)
df_有售商品_3P$第一層 <- str_trim(df_有售商品_3P$第一層)


df_有售商品_1P$`部.課.NEW` <- str_trim(df_有售商品_1P$`部.課.NEW`)
df_有售商品_3P$`部.課.NEW` <- str_trim(df_有售商品_3P$`部.課.NEW`)


```


```{r}

# 啟用多核心與進度條
plan(multisession, workers = parallel::detectCores() - 1)
handlers("txtprogressbar")

start_time <- Sys.time()
all_matches <- list()

# 找出交集：同時出現在 1P 和 3P 中的 部.課.NEW + 第一層
common_keys <- intersect(
  unique(paste(df_有售商品_3P$`部.課.NEW`, df_有售商品_3P$第一層, sep = "___")),
  unique(paste(df_有售商品_1P$`部.課.NEW`, df_有售商品_1P$第一層, sep = "___"))
)

# 依據 部.課.NEW + 第一層 一一比對
for (key in common_keys) {
  key_parts <- strsplit(key, "___")[[1]]
  部課 <- key_parts[1]
  層級 <- key_parts[2]

  cat("🔍 比對部.課.NEW：", 部課, "｜第一層：", 層級, "\n")

  df_3P_layer <- df_有售商品_3P %>% filter(`部.課.NEW` == 部課, 第一層 == 層級)
  df_1P_layer <- df_有售商品_1P %>% filter(`部.課.NEW` == 部課, 第一層 == 層級)

  vec_new <- df_3P_layer$商品名稱
  vec_all <- df_1P_layer$商品名稱

  res <- with_progress({
    p <- progressor(along = vec_new)
    future_lapply(seq_along(vec_new), function(i) {
      p()
      dists <- stringdist(vec_new[i], vec_all, method = "jaccard")
      min_dist <- min(dists, na.rm = TRUE)
      if (min_dist <= 0.3) {
        idx <- which.min(dists)
        data.frame(
          原索引 = df_3P_layer$商品代碼[i],
          商品名稱_3P = vec_new[i],
          商品名稱_比對結果 = vec_all[idx],
          相似度 = 1 - min_dist,
          `部.課.NEW` = 部課,
          第一層 = 層級,
          stringsAsFactors = FALSE
        )
      } else {
        NULL
      }
    })
  })

  all_matches[[key]] <- do.call(rbind, res)
}

plan(sequential)
df_best_match <- do.call(rbind, all_matches)

# 合併原始 3P 商品資料
df_best_match <- df_有售商品_3P %>%
  right_join(df_best_match, by = c("商品代碼" = "原索引"))

# 顯示執行時間
end_time <- Sys.time()
cat("★ 總耗時：", round(as.numeric(difftime(end_time, start_time, units = "mins")), 1), "分鐘\n")

write.xlsx(df_best_match, "商品相似度分析_layer1.xlsx", overwrite = TRUE)

```



```{r}
df_best_match_1 <- df_best_match%>%
  left_join(df_有售商品_1P, by = c("商品名稱_比對結果" = "商品名稱", "第一層" = "第一層","部.課.NEW"="部.課.NEW"))
# :white_check_mark: **儲存結果到 Excel**
write.xlsx(df_best_match_1, "商品相似度分析_layer1.xlsx", overwrite = TRUE)
print(":white_check_mark: 商品相似度分析已完成，結果儲存至 '商品相似度分析.xlsx'！")
```



