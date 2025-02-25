library(readr)
library(readxl)
library(purrr)
library(magrittr)
library(tidyr)
library(dplyr)

# import data -------------------------------------------------------------

list.xlsx.dk <-
  list.files("RawData/KawaijukuHensachi20250210", 
             pattern = "dk", full.names = T) %>% 
  set_names(., nm = .)

list.df.dk <- map(list.xlsx.dk, \(path){read_excel(path)})

list.xlsx.ds <-
  list.files("RawData/KawaijukuHensachi20250210", 
             pattern = "ds", full.names = T) %>% 
  set_names(., nm = .)

list.df.ds <- map(list.xlsx.ds, \(path){read_excel(path)})

# クリーニング関数の定義
# clean_data <- function(df) {
#   df %>%
#     fill(大学名, 学部名, .direction = "down") %>%  # 大学名・学部名の空欄を埋める
#     select(-matches("[/()]"))  # スラッシュや括弧を含む列を削除
# }

# Excelファイルの読み込みとクリーニング
# list.xlsx.dk <- list.files("RawData/KawaijukuHensachi20250210", pattern = "dk", full.names = TRUE)
# list.df.dk <- map(list.xlsx.dk, ~ read_excel(.x) %>% clean_data())

# national and municipal universities (dk) -------------------------------------

#列名の変更
temp_dk <- list.df.dk[[1]] %>%
  rename(
    ボーダー = "ボーダー/満点（得点率）",
    満点 = "...8",      
    得点率 = "...10",    
  )

#数という列の型を変更
temp_dk <-
  temp_dk %>%
  mutate(数 = if_else(数=="－", NA_character_, 数)) %>% 
  mutate(数 = as.numeric(数))  

#temp_dkの7,9,11列目そのものを削除
temp_dk <- temp_dk %>%
  select(-c(`...7`, `...9`, `...11`))

#temp_dkの大学名、日程、学部のNAをNAの上の名称と同じにする
temp_dk <- 
  temp_dk %>%
  fill(c("大学名", "日程", "学部"), .direction = "down")

#private univarsities(ds)-----------------------------------------------

temp_ds <- list.df.ds[[1]] %>%
  rename(
    ボーダー = "ボーダー/満点（得点率）",
    満点　= "...9",
    得点率　= "...11"
  )
#数という列の型を変更
temp_ds <- temp_ds %>%
  mutate(数 = as.numeric(数)) 




#temp_dsの8,10,12列目を削除
temp_ds <- temp_ds %>%
  select(-c(`...8`, `...10`, `...12`))



#temp_dsの大学名、学部のNAをNAの上の名称と同じにする
temp_ds <- temp_ds %>%
  fill(1:2, .direction = "down")

#temp_dsの偏差値のBFを0に置き換える。
temp_ds <-
  temp_ds %>%
  mutate(偏差値　= if_else(偏差値=="BF", 0, 偏差値)) %>%
  mutate(偏差値　= as.numeric(偏差値))


print(temp_ds[5, ])
