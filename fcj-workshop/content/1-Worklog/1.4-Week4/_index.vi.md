---
title: "Worklog Tuần 4"
date: 2026-06-23
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu

Quản lý lưu trữ đám mây với Amazon S3 và thiết lập bảo mật bằng IAM Roles (Identity and Access Management) để các dịch vụ có thể giao tiếp an toàn với nhau mà không cần lưu trữ key.

---

### Nhiệm vụ đã hoàn thành

- [x] Tạo Amazon S3 Bucket để lưu trữ ảnh OCR.
- [x] Tạo IAM Role và gắn Policy cho phép truy cập (Read/Write) vào S3.
- [x] Gắn (Attach) IAM Role vào EC2 instance.
- [x] Viết script/code test thử việc upload ảnh từ EC2 lên S3.

---

### Chi tiết thực hiện

**1. Khởi tạo Amazon S3 Bucket**
- Tạo bucket với tên định danh duy nhất (VD: `my-ocr-images-project`).
- Thiết lập Block Public Access để ngăn chặn lộ dữ liệu nhạy cảm ra ngoài Internet. Dữ liệu chỉ được truy xuất qua ứng dụng.

**2. Cấu hình IAM Role**
Thay vì hardcode `Access Key` và `Secret Key` trong mã nguồn Spring Boot, ta sử dụng IAM Role:
- Tạo IAM Role với trusted entity là **EC2**.
- Viết JSON Policy chỉ cho phép thao tác trên bucket vừa tạo:
```json
{
  "Effect": "Allow",
  "Action": [
    "s3:PutObject",
    "s3:GetObject"
  ],
  "Resource": "arn:aws:s3:::my-ocr-images-project/*"
}
```

**3. Tích hợp IAM Role vào EC2**
- Trở lại màn hình quản lý EC2, chọn instance `t3.micro`.
- Chọn **Security** -> **Modify IAM Role** và gán Role vừa tạo.
- Giờ đây, các container (VD: Spring Boot Backend) chạy bên trong EC2 này có thể tự động kế thừa quyền từ EC2 để gọi AWS API (upload S3) một cách an toàn.

---

### Khó khăn & Giải pháp

| # | Khó khăn | Giải pháp |
|---|----------|-----------|
| 1 | Lỗi Access Denied khi code đẩy ảnh lên S3 | Kiểm tra lại Policy JSON. Quên thêm `/*` ở cuối tên bucket trong phần Resource. (Phải là `arn:aws:s3:::bucket-name/*`) |
| 2 | Sợ lộ dữ liệu OCR khi đưa lên Cloud | Đảm bảo bật Block Public Access cho S3 bucket và cấu hình mã hóa mặc định (SSE-S3). |

---

### Tài liệu tham khảo

- [Amazon S3 Getting Started](https://docs.aws.amazon.com/AmazonS3/latest/userguide/GetStartedWithS3.html)
- [IAM Roles for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
- [Security Best Practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
