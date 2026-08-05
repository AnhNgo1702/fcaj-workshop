---
title: "Worklog Week 4"
date: 2026-06-23
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Objectives

Manage cloud storage with Amazon S3 and set up security using IAM Roles (Identity and Access Management) so that services can communicate securely without storing static access keys.

---

### Completed Tasks

- [x] Create an Amazon S3 Bucket to store OCR images.
- [x] Create an IAM Role and attach a Policy granting Read/Write access to S3.
- [x] Attach the IAM Role to the EC2 instance.
- [x] Write a test script/code to verify image upload from EC2 to S3.

---

### Implementation Details

**1. Amazon S3 Bucket Initialization**
- Create a bucket with a globally unique name (e.g., `my-ocr-images-project`).
- Enable Block Public Access to prevent sensitive data exposure to the Internet. Data will only be accessible via the application.

**2. IAM Role Configuration**
Instead of hardcoding `Access Key` and `Secret Key` inside the Spring Boot source code, we use IAM Roles:
- Create an IAM Role with **EC2** as the trusted entity.
- Write a JSON Policy allowing operations only on the created bucket:
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

**3. Attach IAM Role to EC2**
- Go back to the EC2 management console and select the `t3.micro` instance.
- Go to **Security** -> **Modify IAM Role** and attach the newly created Role.
- Now, containers (e.g., Spring Boot Backend) running inside this EC2 instance can automatically inherit permissions to securely call AWS APIs (like uploading to S3).

---

### Challenges & Solutions

| # | Challenge | Solution |
|---|-----------|----------|
| 1 | Access Denied error when pushing images to S3 | Re-checked the Policy JSON. Forgot to append `/*` to the bucket name in the Resource field. (Must be `arn:aws:s3:::bucket-name/*`) |
| 2 | Concern about OCR data leaks in the Cloud | Ensured Block Public Access is enabled for the S3 bucket and default encryption (SSE-S3) is configured. |

---

### References

- [Amazon S3 Getting Started](https://docs.aws.amazon.com/AmazonS3/latest/userguide/GetStartedWithS3.html)
- [IAM Roles for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
- [Security Best Practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
