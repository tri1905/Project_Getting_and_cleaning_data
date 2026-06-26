---
title: "Quy Trình Tiền Xử Lý Và Đồng Bộ Dữ Liệu Khí Tượng - Năng Suất Cây Trồng Toàn Cầu"
subtitle: "Dự án hoàn thành khóa học JHU - Getting and Cleaning Data"
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

Dữ liệu khí tượng và năng suất nông nghiệp thu thập từ các nguồn mở trực tuyến thường bị phân mảnh, không đồng nhất về định dạng thời gian và tồn tại nhiều sai sót do quá trình nhập liệu thủ công.

**Mục tiêu:** Áp dụng tư duy *Tidy Data* của khóa học JHU để làm sạch, xử lý chuỗi lỗi, và hợp nhất hai nguồn dữ liệu này lại thành một tập dữ liệu chuẩn mực phục vụ cho các mô hình thống kê sinh học phía sau.

## 2. Khai Báo Thư Viện

Chúng ta sử dụng hệ sinh thái `tidyverse` (gồm `dplyr`, `tidyr`, `stringr`, `lubridate`) kết hợp với `Hmisc` để khảo sát cấu trúc dữ liệu.

```{r}
#| label: setup 
library(tidyverse) 
library(readxl) 
library(Hmisc) 
library(lubridate)
```

# Nạp các hàm bổ trợ tự viết từ thư mục scripts

```{r}
source("scripts/cleaning_functions.R")
```

🚀 ĐỀ TÀI GỢI Ý: Phân tích & Làm sạch Dữ liệu Khí tượng Toàn cầu và Đa dạng Cây trồng (Tidy Global Weather & Crop Data) Thay vì dùng dữ liệu giả lập, bạn hãy sử dụng hai nguồn dữ liệu mở, có độ "rác" rất cao trên Internet để thực hành:

Dữ liệu Thời tiết (Khí tượng): Lấy từ dự án TidyTuesday danh tiếng (Tập dữ liệu Weather Data). Dữ liệu này ghi lại nhiệt độ, lượng mưa theo giờ/ngày của nhiều trạm khí tượng, chứa rất nhiều lỗi định dạng ngày tháng, giá trị dị biệt (outliers do hỏng cảm biến) và định dạng dọc-ngang hỗn lộn.

Link lấy data: Bạn có thể tải trực tiếp qua file raw trên GitHub của cộng đồng rfordatascience/tidytuesday.

Dữ liệu Năng suất/Khảo nghiệm: Lấy từ kho dữ liệu mở Kaggle (Tìm kiếm từ khóa: "Crop Yield Prediction Dataset" hoặc "Agricultural Production Namibia/Vietnam"). Tập dữ liệu này thường lưu theo dạng Rộng (Wide format - các năm hoặc các chỉ số là các cột riêng biệt), tên các loại cây hoặc địa danh bị viết sai chính tả hoặc không đồng bộ font chữ.

Các thử thách "Làm sạch" bạn sẽ giải quyết: Xoay trục dữ liệu (tidyr): Đưa bảng năng suất từ dạng Rộng sang dạng Dọc để đạt tiêu chuẩn Tidy Data.

Đồng bộ thời gian (lubridate): Chuyển đổi các chuỗi ngày tháng lộn xộn từ các trạm khí tượng về một kiểu dữ liệu Date đồng nhất.

Xử lý Chuỗi & Regex (stringr): Làm sạch tên các giống cây, loại bỏ khoảng trắng thừa hoặc ký tự đặc biệt.

📂 CẤU TRÚC THƯ MỤC DỰ ÁN QUARTO CHUẨN Khi tạo một dự án mới trong RStudio, bạn hãy tổ chức các file theo sơ đồ dưới đây:

``` text
📦 Project_Getting_and_cleaning_data/          <-- Thư mục gốc (Root - là chính Repo của bạn trên GitHub)
 │
 ├── 📂 data/                           <-- Quản lý toàn bộ dữ liệu dự án
 │    ├── 📂 raw/                       <-- Nơi chứa dữ liệu thô ban đầu (ĐÓNG BẰNG - tuyệt đối không sửa tay)
 │    │    ├── 📄 weather_raw.csv       
 │    │    └── 📄 crop_yield_raw.xlsx   
 │    │
 │    └── 📂 processed/                 <-- Nơi chứa dữ liệu sạch sau khi đã xử lý xong
 │         └── 📄 final_crop_weather_tidy.csv
 │
 ├── 📂 scripts/                        <-- Chứa các file code R thuần (.R) để xử lý tác vụ nặng
 │    ├── 📜 download_data.R            <-- Code tự động tải dữ liệu tự động từ Web/GitHub
 │    └── 📜 cleaning_functions.R       <-- Chứa các hàm dọn dẹp chuỗi, đồng bộ ngày tháng
 │
 ├── 📂 _extensions/                    <-- (Tùy chọn) Chứa các template, filter mở rộng của Quarto
 │
 ├── 📝 _quarto.yml                     <-- FILE CẤU HÌNH CHÍNH (Nơi cài đặt giao diện, mục lục, font chữ...)
 ├── 📊 index.qmd                       <-- FILE BÁO CÁO QUARTO CHÍNH (Nơi viết nội dung, phân tích và render ra HTML)
 ├── 🛠️ crop-weather-project.Rproj     <-- File quản lý không gian làm việc của RStudio
 ├── 📓 README.md                       <-- File giới thiệu tổng quan dự án, cách chạy code (Hiển thị trên GitHub)
 └── ⚙️ .gitignore                       <-- File khai báo các file/thư mục nặng KHÔNG muốn đẩy lên GitHub (ví dụ: data/raw)
```
