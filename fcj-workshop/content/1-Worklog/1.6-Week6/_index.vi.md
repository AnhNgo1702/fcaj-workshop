---
title: "Worklog Tuần 6"
date: 2026-06-23
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu

Triển khai toàn bộ các thành phần ứng dụng (Next.js Frontend, Spring Boot Backend, Hermes Agent) vào các Docker container và kết nối chúng qua Internal API trong cùng một Docker network.

---

### Nhiệm vụ đã hoàn thành

- [x] Đóng gói (Dockerize) Spring Boot Backend và chạy trên Port 8080.
- [x] Đóng gói (Dockerize) Next.js Frontend và chạy trên Port 3000.
- [x] Đóng gói và triển khai Hermes Agent chạy trên Port 5000.
- [x] Cấu hình giao tiếp nội bộ giữa các container thông qua tên của service thay vì IP.
- [x] Thiết lập kết nối Webhook giữa Hermes Agent và Telegram API.

---

### Chi tiết thực hiện

**1. Viết Dockerfile cho từng dịch vụ**
Mỗi dịch vụ cần một `Dockerfile` riêng:
- **Backend (Spring Boot)**: Sử dụng JDK image để build file `.jar` và chạy ứng dụng.
- **Frontend (Next.js)**: Sử dụng Node.js image, chạy `npm run build` và khởi động server.
- **Hermes Agent**: Tùy theo ngôn ngữ (Python/Nodejs), sử dụng image tương ứng.

**2. Tích hợp vào docker-compose.yml**
Mở rộng file `docker-compose.yml` từ Tuần 5:
```yaml
version: '3.8'
services:
  # ... (mysql-db config)
  backend:
    build: ./backend
    container_name: spring-boot-backend
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=mysql-db
    networks:
      - my-app-network
    depends_on:
      - mysql-db

  frontend:
    build: ./frontend
    container_name: nextjs-frontend
    ports:
      - "3000:3000"
    networks:
      - my-app-network

  hermes-agent:
    build: ./hermes
    container_name: hermes-agent
    ports:
      - "5000:5000"
    networks:
      - my-app-network
```

**3. Giao tiếp qua Internal API**
Do các container nằm chung `my-app-network`, chúng có thể gọi nhau thông qua `container_name`. Ví dụ: Spring Boot kết nối database qua host là `mysql-db:3306`, Frontend gọi API Backend qua `http://spring-boot-backend:8080`.

**4. Khởi chạy toàn bộ hệ thống**
Chạy lệnh `docker-compose up -d --build` để build image và khởi động tất cả container cùng lúc.

---

### Khó khăn & Giải pháp

| # | Khó khăn | Giải pháp |
|---|----------|-----------|
| 1 | Backend không kết nối được Database khi khởi động | Database cần thời gian khởi động (khoảng vài giây). Backend lại khởi động quá nhanh. Cấu hình script retry hoặc thêm cơ chế reconnect tự động trong Spring Boot. |
| 2 | EC2 bị treo khi build Docker image | Do `t3.micro` thiếu RAM khi build ứng dụng Node.js hoặc Java. Việc tạo thêm Swap (Tuần 3) đã giải quyết vấn đề này. |

---

### Tài liệu tham khảo

- [Dockerizing a Spring Boot Application](https://spring.io/guides/gs/spring-boot-docker/)
- [Deploying Next.js with Docker](https://nextjs.org/docs/deployment#docker-image)
- [Networking in Compose](https://docs.docker.com/compose/networking/)
