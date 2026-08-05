---
title: "Worklog Tuần 8"
date: 2026-06-23
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu

Thiết lập hệ thống giám sát log và tài nguyên cho EC2, đồng thời tạo cơ chế cảnh báo tự động thông báo cho đội ngũ phát triển (Developers & Operations) khi có sự cố xảy ra.

---

### Nhiệm vụ đã hoàn thành

- [x] Cài đặt và cấu hình CloudWatch Agent trên EC2.
- [x] Đẩy log của Nginx và các Docker container (thông qua awslogs driver) lên CloudWatch Log groups.
- [x] Tạo Metric Filter trên CloudWatch để bắt các dòng log chứa từ khóa `ERROR`.
- [x] Thiết lập CloudWatch Alarms để theo dõi metric `ERROR`.
- [x] Tích hợp Amazon SNS để gửi Email/SMS cảnh báo mỗi khi Alarm bị kích hoạt.

---

### Chi tiết thực hiện

**1. Cài đặt CloudWatch Agent**
- Đảm bảo EC2 IAM Role (đã cấu hình từ Tuần 4) có thêm policy `CloudWatchAgentServerPolicy`.
- Cài đặt agent và cấu hình file JSON để thu thập log từ `/var/log/nginx/error.log` và đẩy lên log group `/aws/ec2/nginx-error`.

**2. Thu thập Docker Logs**
Thay vì để log của Docker nằm rải rác trên máy ảo, cấu hình `docker-compose.yml` sử dụng `awslogs` logging driver:
```yaml
    logging:
      driver: awslogs
      options:
        awslogs-region: ap-southeast-1
        awslogs-group: /aws/docker/spring-boot-backend
        awslogs-create-group: "true"
```
Các log của ứng dụng giờ đây sẽ được stream trực tiếp lên CloudWatch.

**3. Tạo Metric Filter & Alarms**
- Truy cập CloudWatch Log groups, tạo Metric Filter với filter pattern là `"?ERROR" "?Exception"`.
- Tạo Alarm dựa trên Metric vừa tạo, thiết lập điều kiện kích hoạt khi số lượng lỗi lớn hơn 0 trong 1 phút.

**4. Thiết lập Cảnh báo qua SNS**
- Tạo một Amazon SNS Topic tên là `AppErrorAlerts`.
- Thêm Subscription loại **Email** (và SMS nếu cần) cho đội ngũ Developers. Xác nhận subscription qua email.
- Quay lại cấu hình Alarm ở bước 3, phần "Notification", chọn gửi thông báo tới Topic `AppErrorAlerts`.

---

### Khó khăn & Giải pháp

| # | Khó khăn | Giải pháp |
|---|----------|-----------|
| 1 | Lỗi cấp quyền khi Docker đẩy log lên CloudWatch | EC2 IAM Role thiếu quyền tạo log group. Đã cập nhật Policy IAM cấp quyền `logs:CreateLogGroup` và `logs:PutLogEvents`. |
| 2 | Nhận quá nhiều email rác (spam) từ CloudWatch Alarm | Cấu hình Alarm bớt nhạy cảm (VD: > 5 errors trong 5 phút) hoặc tinh chỉnh lại bộ lọc Metric Filter chỉ bắt các lỗi nghiệp vụ nghiêm trọng. |

---

### Tài liệu tham khảo

- [Collect Metrics and Logs with CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
- [Use Amazon CloudWatch Logs with Docker](https://docs.docker.com/config/containers/logging/awslogs/)
- [Amazon SNS Getting Started](https://docs.aws.amazon.com/sns/latest/dg/sns-getting-started.html)
