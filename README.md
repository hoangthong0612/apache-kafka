# Hướng dẫn Cài đặt & Vận hành Apache Kafka qua Docker Compose

Tài liệu này hướng dẫn cách sử dụng tệp `docker-compose.yml` để khởi chạy **Apache Kafka** (chế độ KRaft, không cần ZooKeeper) kết hợp với **Kafka UI** trên máy tính cá nhân hoặc máy chủ phát triển.

---

## 1. Yêu cầu Hệ thống

- Đã cài đặt **Docker** (phiên bản mới nhất).
- Đã cài đặt **Docker Compose** (phiên bản tương thích với định dạng `version: '3.8'`).
- Cổng kết nối `9092` (dành cho Kafka broker) và `8383` (dành cho Kafka UI) phải đang trống.

---

## 2. Cấu trúc Dịch vụ trong `docker-compose.yml`

Hệ thống bao gồm 2 dịch vụ chính chạy trong cùng một mạng Docker ảo:

### 2.1. `kafka` (Apache Kafka - KRaft Mode)
- **Image:** `apache/kafka:latest`
- **Container Name:** `kafka`
- **Cổng ánh xạ (Ports):** `9092:9092` (cho phép các ứng dụng ngoài container kết nối tới Kafka thông qua `localhost:9092`).
- **Cấu hình môi trường cốt lõi:**
  - Sử dụng chế độ **KRaft** (`KAFKA_PROCESS_ROLES: broker,controller`), loại bỏ hoàn toàn sự phụ thuộc vào Apache ZooKeeper.
  - Thiết lập bộ lọc listener nội bộ và ngoại vi rõ ràng giúp giao tiếp thông suốt giữa các container và máy chủ host.
- **Dữ liệu bền vững (Volumes):** Sử dụng volume `kafka_data` gắn vào `/var/lib/kafka/data` để lưu trữ dữ liệu message không bị mất khi restart container.

### 2.2. `kafka-ui` (Provectus Kafka UI)
- **Image:** `provectuslabs/kafka-ui:latest`
- **Container Name:** `kafka-ui`
- **Cổng ánh xạ (Ports):** `8383:8080` (Giao diện web quản lý Kafka chạy tại `http://localhost:8383`).
- **Kết nối:** Tự động kết nối tới cụm Kafka thông qua tên service mạng nội bộ `kafka:29092`.

---

## 3. Hướng dẫn Vận hành

### Bước 1: Khởi động các dịch vụ
Mở terminal tại thư mục chứa tệp `docker-compose.yml` và chạy lệnh sau để khởi động nền (detached mode):

```bash
docker compose up -d
```

### Bước 2: Kiểm tra trạng thái container
Đảm bảo rằng cả hai container `kafka` và `kafka-ui` đang chạy ổn định:

```bash
docker ps
```

### Bước 3: Truy cập Giao diện Quản lý (Kafka UI)
Mở trình duyệt web và truy cập vào đường dẫn:
👉 **[http://localhost:8383](http://localhost:8383)**

Tại đây, bạn có thể:
- Xem tổng quan cụm broker (`local-cluster`).
- Tạo, chỉnh sửa, xóa các **Topics**.
- Theo dõi các **Consumers**, **Producers** và xem trực tiếp nội dung các message trong topic theo thời gian thực.

---

## 4. Các Lệnh Quản lý Thường dùng

- **Xem log thời gian thực của Kafka:**
  ```bash
  docker logs -f kafka
  ```

- **Dừng các dịch vụ (giữ nguyên dữ liệu):**
  ```bash
  docker compose stop
  ```

- **Tắt và xóa các container (giữ nguyên dữ liệu trong volume):**
  ```bash
  docker compose down
  ```

- **Xóa toàn bộ container kèm theo dữ liệu Kafka (Xóa sạch message đã lưu):**
  ```bash
  docker compose down -v
  ```
