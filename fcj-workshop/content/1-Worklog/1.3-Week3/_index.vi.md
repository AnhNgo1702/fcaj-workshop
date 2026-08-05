---
title: "Worklog Tuần 3"
date: 2026-06-23
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu

Cấu hình chi tiết máy chủ EC2 `t3.micro` làm nền tảng chính cho toàn bộ ứng dụng, thiết lập bảo mật mạng và cấp phát tài nguyên bộ nhớ (Swap) để đảm bảo server đủ dung lượng chạy nhiều Docker container.

---

### Nhiệm vụ đã hoàn thành

- [x] Khởi tạo EC2 instance với loại `t3.micro` (1GB RAM).
- [x] Tạo và cấu hình Security Group cho phép truy cập HTTP/HTTPS.
- [x] Kết nối SSH an toàn vào máy chủ.
- [x] Tạo và cấp phát thêm 2GB Swap memory để hỗ trợ RAM.
- [x] Kiểm tra và cập nhật các package hệ điều hành cơ bản.

---

### Chi tiết thực hiện

**1. Khởi tạo EC2 và Security Group**
- Lựa chọn hệ điều hành Amazon Linux 2023 hoặc Ubuntu.
- Tạo Security Group với các Inbound rules:
  - Cho phép `Port 22` từ My IP (để kết nối SSH an toàn).
  - Cho phép `Port 80` (HTTP) và `Port 443` (HTTPS) từ Anywhere (IPv4/IPv6) để Web User truy cập.

**2. Cấu hình Swap Memory**
Do instance `t3.micro` chỉ có 1GB RAM, việc chạy nhiều dịch vụ (Next.js, Spring Boot, MySQL, Hermes Agent) sẽ gây thiếu bộ nhớ (Out-Of-Memory). Do đó, cần cấu hình 2GB Swap file để hệ thống mượn dung lượng ổ cứng làm RAM ảo:
```bash
sudo dd if=/dev/zero of=/swapfile bs=128M count=16
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
# Cấu hình tự động mount swap sau khi reboot
echo '/swapfile swap swap defaults 0 0' | sudo tee -a /etc/fstab
```

**3. Kết nối và Cập nhật hệ thống**
Sử dụng SSH key file (`.pem`) để kết nối an toàn từ máy cá nhân vào EC2:
```bash
ssh -i "my-key.pem" ubuntu@<ec2-public-ip>
sudo apt update && sudo apt upgrade -y
```

---

### Khó khăn & Giải pháp

| # | Khó khăn | Giải pháp |
|---|----------|-----------|
| 1 | Lệnh tạo Swap chạy lâu hoặc báo lỗi | Đảm bảo EBS volume gắn vào EC2 có đủ dung lượng trống (ít nhất 2GB). Lệnh `dd` cần chạy dưới quyền `root` (`sudo`). |
| 2 | EC2 khởi động lại bị mất Swap | Quên bước cấu hình file `/etc/fstab`. Đã bổ sung cấu hình tự động mount swap lúc boot hệ thống. |

---

### Tài liệu tham khảo

- [Amazon EC2 Getting Started](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
- [How to add swap space to Amazon EC2](https://repost.aws/knowledge-center/ec2-memory-swap-file)
- [Amazon EC2 Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)
