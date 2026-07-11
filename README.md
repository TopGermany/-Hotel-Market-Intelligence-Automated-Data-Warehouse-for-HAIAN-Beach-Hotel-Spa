# 🏨 HAIAN HOTEL - DATA WAREHOUSE (DWH) PROJECT

Đây là dự án xây dựng Kho dữ liệu (Data Warehouse) tự động cho Khách sạn Hải An, tập trung vào việc thu thập, làm sạch và phân tích dữ liệu giá phòng của đối thủ cạnh tranh trên thị trường (Booking.com, Trip.com).

## 🚀 1. KIẾN TRÚC HỆ THỐNG
Dự án được xây dựng theo chuẩn kiến trúc Medallion (Đồng - Bạc - Vàng):

1. **Scraper (Lớp Đồng - Bronze):** Sử dụng Playwright để cào dữ liệu giá phòng thực tế của các đối thủ trên OTA mỗi đêm.
2. **ETL Agent (Lớp Bạc - Silver):** Lọc bỏ rác (chống số âm, loại bỏ lỗi 0đ), kiểm tra Overbooking, và đẩy lên Google BigQuery. Nếu có lỗi, tự động bắn Email cảnh báo cho Quản lý.
3. **Data Mart (Lớp Vàng - Gold):** Tạo các View dữ liệu trên BigQuery để kết nối thẳng vào Looker Studio làm Dashboard.
4. **Apache Airflow (Orchestrator):** Vị "nhạc trưởng" điều phối tự động toàn bộ 3 quy trình trên vào lúc 2h00 sáng mỗi ngày.

## 🛠 2. HƯỚNG DẪN CÀI ĐẶT

Dự án yêu cầu cài đặt Docker Desktop trên Windows hoặc môi trường Docker trên Linux.

### Bước 1: Khởi tạo biến môi trường (.env)
Copy file `.env.example` thành `.env` và điền thông tin:
- `GCP_PROJECT_ID`: ID dự án trên Google Cloud.
- `GOOGLE_CREDENTIALS_PATH`: (Để trống nếu chạy trên Airflow Docker, cấu hình service account qua GCP).
- Thông tin SMTP Email để nhận cảnh báo.

### Bước 2: Bật hệ thống Airflow
Mở Terminal / PowerShell ở thư mục gốc của dự án, gõ lệnh:
```bash
docker compose up -d --build
```

### Bước 3: Đăng nhập Bảng điều khiển
- Truy cập trình duyệt: `http://localhost:8080`
- User: `admin` | Pass: `admin`
- Bật công tắc (Toggle) cho DAG `haian_competitor_price_pipeline` sang trạng thái **ON** để nó tự động chạy hàng ngày.

## 📁 3. CẤU TRÚC THƯ MỤC CHÍNH

- `/scrapers/`: Mã nguồn bot cào dữ liệu (Trip.com, Booking.com).
- `/agents/`: Chứa file `competitor_scraper_agent.py` và `etl_agent.py`.
- `/dags/`: Chứa kịch bản điều phối của Airflow (`haian_pipeline_dag.py`).
- `/Data/`: Thư mục lưu trữ file CSV tải về (Local Storage).
- **Muốn chỉnh sửa Data Mart:** Mở file `agents/create_data_marts.py` để viết thêm các lệnh SQL tạo View mới trên BigQuery.

## 🟢 5. QUY TRÌNH BẬT / TẮT AN TOÀN (TIẾT KIỆM RAM)

Vì Airflow và Docker (vmmemWSL) ngốn rất nhiều RAM của máy tính. Nếu bạn chỉ chạy trên máy tính cá nhân (Laptop), hãy tuân thủ quy trình sau để máy không bị lag:

### Quy trình BẬT (Khi cần làm việc):
1. Mở phần mềm **Docker Desktop** lên. (Thao tác này sẽ tự động đánh thức lõi máy ảo `vmmemWSL`). Đợi góc trái dưới hiện chữ `Engine Running` màu xanh lá.
2. Mở Terminal / PowerShell tại thư mục dự án, gõ lệnh:
   ```bash
   docker compose up -d
   ```
3. Mở trình duyệt web truy cập `http://localhost:8080` (admin/admin).

### Quy trình TẮT (Khi xong việc, muốn giải phóng 100% RAM):
1. Mở Terminal / PowerShell tại thư mục dự án, gõ lệnh để tắt Airflow:
   ```bash
   docker compose down
   ```
2. Tắt hoàn toàn phần mềm Docker Desktop (Click chuột phải vào icon hình con cá voi ở góc phải dưới màn hình $\rightarrow$ Chọn **Quit Docker Desktop**).
3. Mở Terminal gõ lệnh sau để "ép" máy ảo Linux nhả RAM lại cho Windows:
   ```bash
   wsl --shutdown
   ```
   *(Lúc này kiểm tra Task Manager sẽ thấy vmmemWSL biến mất hoàn toàn).*
