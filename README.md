---
title: "Quy Trình Tiền Xử Lý Và Chuẩn Hóa Dữ Liệu Mở TidyTuesday"
subtitle: "Dự án hoàn thành khóa học JHU - Getting and Cleaning Data (Tuần 17/02/2026)"
author: "Cao Minh Trí"
date: today
format:
  html:
    toc: true
    toc-depth: 3
    code-fold: true
    code-tools: true
    theme: cosmo
execute:
  warning: false
  message: false
---

## 1. Đặt Vấn Đề & Mục Tiêu Nghiên Cứu

Dữ liệu thực tế được chia sẻ từ cộng đồng mở trực tuyến thường tồn tại nhiều lỗi hệ thống như cấu trúc bảng chưa tối ưu, dữ liệu khuyết thiếu (`NA`), xung đột định dạng thời gian và nhiễu ký tự do nhập liệu thủ công.

**Mục tiêu:** Áp dụng tư duy *Tidy Data* cốt lõi từ khóa học JHU để tiến hành kiểm toán cấu trúc, bẫy lỗi định dạng, làm sạch chuỗi văn bản rác và đóng gói tập dữ liệu của tuần `17/02/2026` về dạng chuẩn mực, sẵn sàng cho các mô hình phân tích thống kê phía sau.

## 2. Khai Báo Thư Viện & Cấu Trúc Dự Án

Chúng ta sử dụng hệ sinh thái `tidyverse` kết hợp với các package bổ trợ để khảo sát sâu và tải dữ liệu một cách tự động.

``` r
#| label: setup
library(tidyverse)
library(Hmisc)
library(lubridate)

# Tự động cài đặt tidytuesdayR nếu hệ thống chưa có
if (!requireNamespace("tidytuesdayR", quiet = TRUE)) install.packages("tidytuesdayR")
library(tidytuesdayR)
```

### 📂 Cấu Trúc Thư Mục Dự Án Quarto Chuẩn

Để dự án đạt tiêu chuẩn nghiên cứu có thể tái lập (Reproducible Research), toàn bộ các file được tổ chức cấu trúc như sau:

```{=texinfo}
tidytuesday-project/
├── tidytuesday-project.Rproj     <- File Project của RStudio
├── README.md                     <- Giới thiệu tổng quan dự án và cách chạy
│
├── data/                         <- Quản lý dữ liệu (Tuyệt đối không sửa tay)
│   ├── raw/                      <- Nơi lưu dữ liệu gốc từ hệ thống (ĐÓNG BĂNG)
│   └── processed/                <- Nơi lưu dữ liệu phẳng sau khi làm sạch
│       └── final_cleaned_data.csv
│
└── index.qmd                     <- FILE BÁO CÁO QUARTO CHÍNH (File này)
```
