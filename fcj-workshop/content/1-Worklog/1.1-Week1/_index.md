---
title: "Week 1 Worklog"
date: 2026-06-23
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Objective

Set up the AWS working environment: create an account, secure the root account, install the AWS CLI, and practice basic operations with EC2.

---

### Tasks Completed

- [x] Create AWS Free Tier account
- [x] Enable MFA on root account
- [x] Create IAM admin user; stop using root for daily tasks
- [x] Install and configure AWS CLI (Access Key, Secret Key, Region)
- [x] Explore the AWS Management Console and main service categories
- [x] Launch first EC2 instance, connect via SSH, and verify the system

---

### Implementation Details

**1. Account Setup**
Created an AWS Free Tier account at [aws.amazon.com](https://aws.amazon.com). Enabled MFA on the root account using an authenticator app (Google Authenticator). Created an IAM user `admin` with `AdministratorAccess`, which is used for all subsequent tasks.

**2. AWS CLI Installation**
Downloaded AWS CLI v2 and configured it with:
```bash
aws configure
# AWS Access Key ID: <key>
# AWS Secret Access Key: <secret>
# Default region name: ap-southeast-1
# Default output format: json
```
Verified connectivity: `aws sts get-caller-identity`

**3. Launch EC2**
Launched a `t2.micro` instance with the Amazon Linux 2023 AMI. Configured the Security Group to allow port `22` from a personal IP only. Connected via SSH:
```bash
ssh -i my-key.pem ec2-user@<public-ip>
```

---

### Challenges & Solutions

| # | Challenge | Solution |
|---|-----------|----------|
| 1 | Unsure which Region to use for the lab | Chose `ap-southeast-1` (Singapore) for low latency from Vietnam |
| 2 | `aws configure` returned invalid credentials error | Inspected the Access Key and found a trailing space. Regenerated the key |

---

### References

- [AWS Getting Started Guide](https://aws.amazon.com/getting-started/)
- [AWS CLI Installation](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Amazon EC2 Getting Started](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)

---

{{% notice info %}}
Never use the root account for daily operations. Create an IAM user with only the permissions needed, and enable MFA on both the root and IAM admin accounts.
{{% /notice %}}
