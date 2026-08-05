---
title: "Worklog Tuần 2"
date: 2026-06-23
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu

Triển khai web server trên EC2 với Nginx và sử dụng S3 lưu trữ tài nguyên tĩnh. Đảm bảo EC2 truy cập S3 qua VPC Endpoint, không đi qua Internet công khai.

---

### Nhiệm vụ đã hoàn thành

- [x] Khởi tạo EC2 `t2.micro` Ubuntu Server 22.04 LTS
- [x] Cài đặt và cấu hình Nginx
- [x] Tạo S3 bucket lưu tài nguyên tĩnh (ảnh, CSS)
- [x] Cấu hình S3 Gateway VPC Endpoint
- [x] Tạo IAM Role `AmazonS3ReadOnlyAccess`, attach vào EC2
- [x] Kiểm tra trang web truy cập thành công qua Public IP

---

### Chi tiết thực hiện

**1. Cài đặt Nginx**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx -y
sudo systemctl enable --now nginx
```

**2. Tạo S3 Bucket**
Tạo bucket cùng Region với EC2. Upload tài nguyên. Giữ chế độ `Block all public access` — EC2 sẽ truy cập qua VPC Endpoint nội bộ.

**3. Cấu hình VPC Endpoint**
Tạo S3 Gateway Endpoint trong VPC, liên kết với Route Table của subnet chứa EC2. Lưu lượng S3 không còn đi ra Internet.

**4. IAM Role cho EC2**
Tạo Role với policy `AmazonS3ReadOnlyAccess`. Attach vào instance qua: **Actions > Security > Modify IAM role**. Xác nhận:
```bash
aws s3 cp s3://<bucket-name>/image.png /var/www/html/
```

---

### Khó khăn & Giải pháp

| # | Khó khăn | Giải pháp |
|---|----------|-----------|
| 1 | `aws s3 cp` báo lỗi Access Denied | EC2 chưa có IAM Role. Tạo Role với S3ReadOnly và attach vào instance |
| 2 | Nginx trả `403 Forbidden` cho file ảnh | File thiếu quyền đọc của `www-data`. Chạy `sudo chmod 644 /var/www/html/image.png` |

---

### Tài liệu tham khảo

- [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/)
- [Amazon S3 Getting Started](https://docs.aws.amazon.com/AmazonS3/latest/userguide/GetStartedWithS3.html)
- [VPC Endpoints for S3](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)
- [IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)

---

{{% notice info %}}
Luôn gán IAM Role cho EC2 thay vì hardcode Access Key vào máy chủ. Hardcode credentials là rủi ro bảo mật nghiêm trọng.
{{% /notice %}}
