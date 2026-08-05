---
title: "Worklog Tuần 7"
date: 2026-06-23
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu

Cài đặt Nginx làm Reverse Proxy để điều hướng traffic từ bên ngoài vào các Docker container tương ứng, đồng thời cấu hình DNS với Amazon Route 53 và thiết lập HTTPS.

---

### Nhiệm vụ đã hoàn thành

- [x] Cài đặt Nginx trên EC2 (cài trực tiếp trên Host, không dùng Docker).
- [x] Cấu hình Nginx Reverse Proxy phân luồng request (`/api/*` tới Backend, `/` tới Frontend).
- [x] Đăng ký tên miền và cấu hình Hosted Zone trên Amazon Route 53.
- [x] Tạo DNS Record trỏ domain về public IP của EC2.
- [x] Đăng ký và cài đặt chứng chỉ SSL/TLS miễn phí qua Let's Encrypt (Certbot) cho Nginx.

---

### Chi tiết thực hiện

**1. Cài đặt và cấu hình Nginx**
- Cài đặt Nginx trên EC2:
```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```
- Tạo file cấu hình `/etc/nginx/conf.d/myapp.conf`:
```nginx
server {
    listen 80;
    server_name mydomain.com;

    # Chuyển hướng Frontend
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Chuyển hướng Backend API
    location /api/ {
        proxy_pass http://localhost:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
- Khởi động lại Nginx: `sudo systemctl restart nginx`.

**2. Cấu hình Amazon Route 53**
- Trong Route 53 console, tạo Hosted Zone cho domain đang sở hữu.
- Tạo một bản ghi (Record) loại **A**, dán địa chỉ Public IPv4 của máy chủ EC2 vào ô Value.
- Chờ DNS cập nhật (propagation).

**3. Thiết lập HTTPS bằng Certbot**
Để bảo vệ dữ liệu (đặc biệt khi gọi API và webhook Telegram), ta cài SSL:
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d mydomain.com
```
Certbot sẽ tự động sửa file cấu hình Nginx để thêm cấu hình SSL trên port 443 và tạo script tự động gia hạn.

---

### Khó khăn & Giải pháp

| # | Khó khăn | Giải pháp |
|---|----------|-----------|
| 1 | Truy cập domain báo lỗi 502 Bad Gateway | Kiểm tra lại log Nginx (`/var/log/nginx/error.log`). Nguyên nhân do Backend container chưa chạy. Chạy lại `docker-compose up -d`. |
| 2 | Lỗi Webhook Telegram không hoạt động | Telegram yêu cầu webhook phải dùng HTTPS. Sau khi cài Certbot SSL ở bước 3, webhook đã hoạt động bình thường. |

---

### Tài liệu tham khảo

- [NGINX Reverse Proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
- [Amazon Route 53 Getting Started](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/getting-started.html)
- [Certbot Instructions for Nginx](https://certbot.eff.org/instructions)
