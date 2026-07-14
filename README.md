# 🏨 Hotel Market Intelligence & Automated Data Warehouse for HAIAN Beach Hotel & Spa

![Data Engineering & Analytics Architecture](https://img.shields.io/badge/Architecture-Medallion-blue)
![Tech Stack](https://img.shields.io/badge/Stack-Python%20%7C%20Airflow%20%7C%20BigQuery-success)
![Data Analyst](https://img.shields.io/badge/Role-Data%20Analyst%20%2B%20Data%20Engineer-orange)

## 📌 1. Tổng quan Dự án & Bài toán Kinh doanh
Đây là một dự án Data Engineering & Analytics (từ A đến Z) được thiết kế nhằm giải quyết bài toán cốt lõi về Quản trị Doanh thu (Revenue Management) cho **Khách sạn HAIAN Beach Hotel & Spa (Đà Nẵng)**.

**Vấn đề Kinh doanh:** Trong mùa cao điểm du lịch (tháng 6 – tháng 8), chỉ số **ProfitPAR (Lợi nhuận trên mỗi phòng khả dụng)** của khách sạn bị sụt giảm xuống mức 550.000 VNĐ/phòng, thấp hơn đáng kể so với mục tiêu kỳ vọng là 850.000 VNĐ/phòng. Mặc dù doanh thu và công suất phòng (Occupancy) cao, nhưng lợi nhuận thực tế lại bị bào mòn do chiến lược giá chưa tối ưu, chi phí hoa hồng OTA (Booking, Trip.com) quá cao và sự cạnh tranh khốc liệt tại khu vực biển Mỹ Khê.

**Giải pháp:** Xây dựng một Kho dữ liệu tự động (Data Warehouse) kết hợp dữ liệu đặt phòng/chi phí nội bộ của khách sạn với dữ liệu giá của đối thủ cạnh tranh được thu thập từ các OTA. Hệ thống này giúp khách sạn chuyển đổi từ *định giá theo kinh nghiệm cảm tính* sang **Mô hình Định giá Động dựa trên Dữ liệu (Data-Driven Dynamic Pricing)**, với mục tiêu khôi phục ProfitPAR đạt ít nhất 765.000 VNĐ trong mùa cao điểm sắp tới.

---

## 🔄 2. Luồng xử lý Dữ liệu (Data Pipeline Workflow)
*(Tổng quan về cách dữ liệu được luân chuyển từ nguồn đến Dashboard)*

<!-- 📸 THÊM ẢNH SƠ ĐỒ WORKFLOW/PIPELINE VÀO DÒNG BÊN DƯỚI -->
<img width="1774" height="887" alt="f17ac0ee-86e2-4667-908e-d382a0c377ab" src="https://github.com/user-attachments/assets/945eea2b-0eb9-462a-926d-98c8d38d34f8" />


Hệ thống hoạt động tự động hàng ngày mà không cần can thiệp thủ công:
1. **Extract (Trích xuất):** Các bot Playwright tự động cào (scrape) giá phòng của đối thủ từ OTA.
2. **Validate (Kiểm tra):** ETL Agent (viết bằng Python) áp dụng các luật kiểm tra chất lượng dữ liệu (Data Quality) nghiêm ngặt.
3. **Load (Tải - UPSERT):** Dữ liệu sạch được hợp nhất vào các bảng trong BigQuery, đảm bảo không bị trùng lặp.
4. **Transform (Chuyển đổi):** Các SQL Views trong BigQuery tổng hợp dữ liệu từ tầng Silver thành các Data Marts ở tầng Gold.
5. **Visualize (Trực quan hóa):** Looker Studio kết nối trực tiếp với Data Marts để render các Dashboard động.

---

## 🛠️ 3. Kiến trúc Kỹ thuật (Data Engineering)
Dự án được xây dựng dựa trên kiến trúc hiện đại **Medallion Architecture (Bronze ➔ Silver ➔ Gold)**:

1. **Tầng Bronze (Ingestion):** 
   - Xây dựng Headless Web Scrapers bằng `Playwright` để thu thập dữ liệu giá trực tiếp từ các nền tảng OTA.
2. **Tầng Silver (Processing & Data Quality):** 
   - Phát triển Python ETL Agent thực hiện kiểm tra Khóa ngoại (Foreign Keys) dựa trên custom Schema Registry.
   - Áp dụng các Business Rules (Luật kinh doanh): Loại bỏ dữ liệu bẩn (như doanh thu âm, bán 0 phòng, phát hiện overbooking) và tích hợp hệ thống cảnh báo qua Email (SMTP).
   - Áp dụng logic **UPSERT** để tải dữ liệu sạch vào **Google BigQuery**, đảm bảo tính toàn vẹn và không trùng lặp dữ liệu.
3. **Tầng Gold (Data Marts):** 
   - Viết các mã BigQuery SQL Views để tổng hợp dữ liệu Silver thành các mô hình phân tích (`mart_profitpar_daily`, `mart_competitor_pricing_summary`, `mart_cost_analysis_daily`).
4. **Điều phối & Tự động hóa (Orchestration):** 
   - Đóng gói toàn bộ Pipeline bằng **Docker**.
   - Lên lịch chạy tự động hàng ngày lúc 02:00 AM bằng **Apache Airflow**.

---

## 🗄️ 4. Mô hình hóa Dữ liệu (ERD - Entity Relationship Diagram)
Để tổ chức Data Warehouse một cách hiệu quả, dữ liệu được mô hình hóa theo **Star Schema (Mô hình sao)**. Thiết kế này giúp tối ưu hóa hiệu suất truy vấn trong BigQuery và cực kỳ trực quan khi kéo thả báo cáo trên Looker Studio.

<!-- 📸 THÊM ẢNH ERD VÀO DÒNG BÊN DƯỚI -->
<img width="2110" height="2160" alt="Quan_ly_TK" src="https://github.com/user-attachments/assets/35382851-6443-4c72-81bf-824d4780c97d" />


**Các thành phần chính:**
- **Bảng Fact (Sự kiện):** `fact_booking`, `fact_room_revenue_daily`, `fact_room_cost_daily`, `fact_competitor_price_snapshot`. Lưu trữ các chỉ số định lượng (doanh thu, chi phí, giá đối thủ).
- **Bảng Dimension (Chiều):** `dim_date`, `dim_hotel`, `dim_room_type`, `dim_channel`, `dim_guest_segment`. Cung cấp ngữ cảnh (ai, cái gì, khi nào, ở đâu) để lọc và phân tích dữ liệu.

---

## 📁 5. Cấu trúc Thư mục (Repository Structure)
```text
├── Data/                   # Lưu trữ dữ liệu Raw cào được (CSV)
├── scrapers/               # Playwright bots cho Booking.com & Trip.com
├── agents/                 # Logic ETL, Schema Registry, Data Quality Checks
│   ├── etl_agent.py        # Logic lõi cho UPSERT và Validation
│   ├── create_data_marts.py# Script SQL tạo tầng Gold (Views) trên BigQuery
│   └── haian_dwh_agent/    # Điều phối thông minh bằng Google ADK
├── dags/                   # Apache Airflow DAGs để lên lịch chạy
├── docker-compose.yml      # Cấu hình khởi tạo Docker container
└── README.md
```

---

## 💡 6. Kỹ năng Ứng dụng
* **Phân tích Dữ liệu (Data Analytics):** SQL (BigQuery), Business Intelligence (Looker Studio), Thiết kế KPI (ProfitPAR, RevPAR, ADR), Revenue Management, Data Modeling.
* **Kỹ sư Dữ liệu (Data Engineering):** Thiết kế ETL Pipeline, Python (Pandas), Data Quality Assurance, Incremental Load (UPSERT).
* **Công cụ & Nền tảng:** Google BigQuery, Apache Airflow, Docker, Playwright, Git.

---

## ⚙️ 7. Hướng dẫn cài đặt & Chạy dự án
*(Dành cho nhà tuyển dụng & kỹ sư đánh giá)*
1. Clone repository này về máy và cấu hình file `.env` với GCP Credentials.
2. Chạy lệnh `docker compose up -d --build` để khởi động môi trường Airflow và Docker.
3. Truy cập giao diện Airflow tại `http://localhost:8080` và trigger DAG `haian_competitor_price_pipeline`.
4. Kiểm tra Google BigQuery để xem Dataset và Data Marts đã được nạp dữ liệu.

---

## 📊 8. Hệ thống Dashboard Phân tích (Looker Studio)

Để chuyển đổi dữ liệu thô thành những thông tin chi tiết có giá trị hành động, tôi đã thiết kế 3 Dashboard Quản trị kết nối trực tiếp với các Data Marts trên BigQuery.

### 💰 8.1. Dashboard Quản trị Doanh thu (Revenue Management)
<!-- 📸 THÊM ẢNH REVENUE DASHBOARD VÀO DÒNG BÊN DƯỚI -->
<img width="1152" height="691" alt="image" src="https://github.com/user-attachments/assets/fa99c41f-a131-47ba-aec2-96a3119e3638" />


**Phân tích & Insight:** Phân rã doanh thu theo Kênh (Channel) và Phân khúc Khách hàng (Customer Segment). Giúp phát hiện nhanh tình trạng nếu Công suất phòng (Occupancy) rất cao nhưng RevPAR lại thấp, báo hiệu rằng khách sạn đang bán phòng với giá quá rẻ.

### 📈 8.2. Dashboard Phân tích Lợi nhuận (Profitability Management)
<!-- 📸 THÊM ẢNH PROFITABILITY DASHBOARD VÀO DÒNG BÊN DƯỚI -->
<img width="1155" height="695" alt="image" src="https://github.com/user-attachments/assets/87a986dd-b277-4680-8ed1-3fb6ae3353b4" />


**Phân tích & Insight:** Trực quan hóa dòng chảy (Waterfall) từ Doanh thu gộp đến Lợi nhuận ròng. Trả lời câu hỏi sống còn: *"Liệu các chương trình voucher và tiền hoa hồng OTA có đang "ăn" hết lợi nhuận của chúng ta không?"* Dashboard giúp xác định chính xác ProfitPAR thực tế mang lại từ từng nguồn đặt phòng.

### 🕵️ 8.3. Dashboard Tình báo Thị trường (Market Intelligence)
<!-- 📸 THÊM ẢNH MARKET INTELLIGENCE DASHBOARD VÀO DÒNG BÊN DƯỚI -->
<img width="1155" height="691" alt="image" src="https://github.com/user-attachments/assets/321ac195-c864-4ad6-82cb-d1c3a3763e91" />


**Phân tích & Insight:** So sánh ADR (Giá bán bình quân) của HAIAN với giá trung bình của thị trường xung quanh. Hệ thống trực quan hóa cảnh báo ngay lập tức khi đối thủ giảm giá mạnh hoặc hết phòng (Sold-out). Từ đó, khách sạn có thể áp dụng chiến lược **Định giá Động (Dynamic Pricing)** một cách linh hoạt.

---

## 🎯 9. Đề xuất Kinh doanh (Actionable Insights)
Dựa trên dữ liệu được thu thập và phân tích từ hệ thống Dashboard, dưới đây là các đề xuất chiến lược nhằm khôi phục ProfitPAR về mức mục tiêu 850.000 VNĐ:

1. **Áp dụng Định giá Động (Dynamic Pricing) theo Chỉ số Cạnh tranh:** Ngừng phụ thuộc vào bảng giá cố định theo mùa. Khi Dashboard Market Intelligence hiển thị đối thủ đã hết phòng (nguồn cung thị trường khan hiếm), HAIAN cần lập tức tăng giá bán (ADR) cho các phòng còn lại.
2. **Tối ưu hóa Kênh Phân phối để Giảm chi phí OTA:** Dashboard Lợi nhuận chỉ ra rằng doanh thu gộp từ OTA rất cao nhưng lợi nhuận ròng lại thấp do mức hoa hồng 15-20%. Khách sạn cần triển khai các chiến dịch đẩy mạnh đặt phòng trực tiếp (Direct Booking) qua website đối với những tệp khách hàng mang lại lợi nhuận cao nhất.
3. **Đánh giá lại Chiến lược Phát hành Voucher:** Việc giảm giá ồ ạt qua Voucher trong mùa cao điểm đang bào mòn Biên lợi nhuận (Profit Margin) mà không cải thiện RevPAR. Ban quản lý cần chuyển sang hình thức giảm giá có điều kiện (Ví dụ: lưu trú tối thiểu 3 đêm) thay vì giảm giá đại trà.
