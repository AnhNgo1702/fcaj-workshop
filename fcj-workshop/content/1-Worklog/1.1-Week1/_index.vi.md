---
title: "Worklog Tuần 1"
date: 2026-06-23
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu

Thiết lập môi trường làm việc trên AWS: tạo tài khoản, bảo mật tài khoản gốc, cài đặt AWS CLI và thực hành các thao tác cơ bản với EC2.

---

### Nhiệm vụ đã hoàn thành

- [x] Tạo AWS Free Tier account
- [x] Bật MFA cho tài khoản root
- [x] Tạo IAM admin user, không dùng root cho công việc hàng ngày
- [x] Cài đặt và cấu hình AWS CLI (Access Key, Secret Key, Region)
- [x] Khám phá AWS Management Console và các nhóm dịch vụ chính
- [x] Tạo EC2 instance đầu tiên, kết nối SSH và kiểm tra hệ thống

---

### Chi tiết thực hiện

**1. Thiết lập tài khoản**
Tạo tài khoản AWS Free Tier tại [aws.amazon.com](https://aws.amazon.com). Bật MFA cho root account bằng ứng dụng xác thực (Google Authenticator). Tạo IAM user `admin` với quyền `AdministratorAccess`, sử dụng user này cho mọi tác vụ tiếp theo.

**2. Cài đặt AWS CLI**
Tải AWS CLI v2 và cấu hình với lệnh:
```bash
aws configure
# AWS Access Key ID: <key>
# AWS Secret Access Key: <secret>
# Default region name: ap-southeast-1
# Default output format: json
```
Kiểm tra kết nối: `aws sts get-caller-identity`

**3. Khởi tạo EC2**
Tạo EC2 `t2.micro` với AMI Amazon Linux 2023. Cấu hình Security Group cho phép port `22` từ IP cá nhân. Kết nối SSH:
```bash
ssh -i my-key.pem ec2-user@<public-ip>
```

---

### Khó khăn & Giải pháp

| # | Khó khăn | Giải pháp |
|---|----------|-----------|
| 1 | Không biết chọn Region nào để làm việc | Chọn `ap-southeast-1` (Singapore) vì gần Việt Nam, độ trễ thấp |
| 2 | Lệnh `aws configure` báo lỗi invalid credentials | Kiểm tra lại Access Key; phát hiện đã copy thừa dấu cách. Tạo lại key mới |

---

### Tài liệu tham khảo

- [AWS Getting Started Guide](https://aws.amazon.com/getting-started/)
- [AWS CLI Installation](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Amazon EC2 Getting Started](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)

---

{{% notice info %}}
Không bao giờ sử dụng root account cho công việc hàng ngày. Tạo IAM user với đúng quyền cần thiết và bật MFA cho cả root lẫn IAM admin user.
{{% /notice %}}
