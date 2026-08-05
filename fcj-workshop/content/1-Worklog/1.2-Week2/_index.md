---
title: "Week 2 Worklog"
date: 2026-06-23
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Objective

Deploy a static website on EC2 using Nginx, with image assets stored in S3. Ensure EC2 accesses S3 via a VPC Endpoint without traversing the public Internet.

---

### Tasks Completed

- [x] Launch EC2 `t2.micro` with Ubuntu Server 22.04 LTS
- [x] Install and configure Nginx
- [x] Create an S3 bucket for static assets (images, CSS)
- [x] Configure an S3 Gateway VPC Endpoint
- [x] Create IAM Role with `AmazonS3ReadOnlyAccess` and attach to EC2
- [x] Verify the website is accessible via the EC2 Public IP

---

### Implementation Details

**1. Install Nginx**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx -y
sudo systemctl enable --now nginx
```

**2. Create S3 Bucket**
Created the bucket in the same Region as EC2. Uploaded assets. Kept `Block all public access` enabled — EC2 accesses the bucket privately via the VPC Endpoint.

**3. Configure VPC Endpoint**
Created an S3 Gateway Endpoint in the VPC and associated it with the Route Table of the EC2 subnet. S3 traffic no longer routes through the Internet.

**4. IAM Role for EC2**
Created an IAM Role with `AmazonS3ReadOnlyAccess` policy. Attached to the instance via: **Actions > Security > Modify IAM role**. Validated:
```bash
aws s3 cp s3://<bucket-name>/image.png /var/www/html/
```

---

### Challenges & Solutions

| # | Challenge | Solution |
|---|-----------|----------|
| 1 | `aws s3 cp` returned Access Denied | EC2 had no IAM Role. Created a Role with S3ReadOnly policy and attached it to the instance |
| 2 | Nginx returned `403 Forbidden` for image files | Missing read permissions for `www-data`. Ran `sudo chmod 644 /var/www/html/image.png` |

---

### References

- [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/)
- [Amazon S3 Getting Started](https://docs.aws.amazon.com/AmazonS3/latest/userguide/GetStartedWithS3.html)
- [VPC Endpoints for S3](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)
- [IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)

---

{{% notice info %}}
Always assign an IAM Role to EC2 instead of hardcoding Access Keys on the server. Hardcoded credentials are a critical security risk.
{{% /notice %}}
