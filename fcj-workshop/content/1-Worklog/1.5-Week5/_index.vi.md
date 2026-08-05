---
title: "Worklog Tuần 5"
date: 2026-06-23
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu

Thiết lập môi trường containerization trên EC2 bằng Docker và Docker Compose, tạo mạng nội bộ (bridge network) và triển khai cơ sở dữ liệu MySQL chạy độc lập trong container.

---

### Nhiệm vụ đã hoàn thành

- [x] Cài đặt Docker và Docker Compose trên Amazon Linux 2023 / Ubuntu.
- [x] Cấu hình phân quyền để user không cần dùng `sudo` khi chạy lệnh docker.
- [x] Tạo Docker network loại `bridge` để các container có thể giao tiếp qua tên miền cục bộ.
- [x] Triển khai MySQL container thông qua file `docker-compose.yml`.
- [x] Kết nối và kiểm tra database từ bên trong EC2.

---

### Chi tiết thực hiện

**1. Cài đặt Docker & Docker Compose**
- Chạy các lệnh cập nhật và cài đặt Docker engine:
```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable docker
sudo systemctl start docker
# Cấp quyền cho user mặc định (ubuntu)
sudo usermod -aG docker ubuntu
```
Sau đó đăng xuất và đăng nhập lại để quyền có hiệu lực.

**2. Khởi tạo Docker Network**
- Tạo một mạng riêng (bridge) để MySQL, Spring Boot và Next.js giao tiếp an toàn, không bị expose bừa bãi:
```bash
docker network create my-app-network
```

**3. Triển khai MySQL Container**
- Tạo thư mục chứa data và file cấu hình `docker-compose.yml`:
```yaml
version: '3.8'
services:
  mysql-db:
    image: mysql:8.0
    container_name: mysql-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: my-secret-pw
      MYSQL_DATABASE: my_app_db
    ports:
      - "3306:3306"
    networks:
      - my-app-network
    volumes:
      - db_data:/var/lib/mysql

networks:
  my-app-network:
    external: true

volumes:
  db_data:
```
- Khởi chạy container ở chế độ nền (detached mode): `docker-compose up -d`.

---

### Khó khăn & Giải pháp

| # | Khó khăn | Giải pháp |
|---|----------|-----------|
| 1 | Lỗi `permission denied` khi chạy lệnh `docker ps` | Do chưa đăng xuất (logout) rồi đăng nhập lại sau khi add user vào group `docker`. |
| 2 | MySQL container bị stop ngay sau khi chạy | Thường do thiếu biến môi trường bắt buộc (như `MYSQL_ROOT_PASSWORD`). Đã bổ sung vào file compose và chạy lại. |

---

### Tài liệu tham khảo

- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Docker Compose Overview](https://docs.docker.com/compose/)
- [Docker Hub - MySQL Official Image](https://hub.docker.com/_/mysql)
