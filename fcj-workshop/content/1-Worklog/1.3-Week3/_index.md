---
title: "Worklog Week 3"
date: 2026-06-23
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Objectives

Configure the `t3.micro` EC2 instance in detail to serve as the main platform for the entire application, set up network security, and allocate memory resources (Swap) to ensure the server has enough capacity to run multiple Docker containers.

---

### Completed Tasks

- [x] Launch a `t3.micro` EC2 instance (1GB RAM).
- [x] Create and configure a Security Group to allow HTTP/HTTPS access.
- [x] Securely connect to the server via SSH.
- [x] Create and allocate an additional 2GB of Swap memory to supplement RAM.
- [x] Check and update basic operating system packages.

---

### Implementation Details

**1. EC2 Initialization and Security Group**
- Select the operating system (Amazon Linux 2023 or Ubuntu).
- Create a Security Group with Inbound rules:
  - Allow `Port 22` from My IP (for secure SSH connection).
  - Allow `Port 80` (HTTP) and `Port 443` (HTTPS) from Anywhere (IPv4/IPv6) for Web User access.

**2. Swap Memory Configuration**
Since the `t3.micro` instance only has 1GB of RAM, running multiple services (Next.js, Spring Boot, MySQL, Hermes Agent) can lead to Out-Of-Memory errors. Therefore, we configure a 2GB Swap file to let the system borrow disk space as virtual RAM:
```bash
sudo dd if=/dev/zero of=/swapfile bs=128M count=16
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
# Automount swap after reboot
echo '/swapfile swap swap defaults 0 0' | sudo tee -a /etc/fstab
```

**3. Connection and System Update**
Use the SSH key file (`.pem`) to securely connect from the local machine to the EC2 instance:
```bash
ssh -i "my-key.pem" ubuntu@<ec2-public-ip>
sudo apt update && sudo apt upgrade -y
```

---

### Challenges & Solutions

| # | Challenge | Solution |
|---|-----------|----------|
| 1 | Swap creation command takes long or fails | Ensure the attached EBS volume has enough free space (at least 2GB). The `dd` command must be run with `root` privileges (`sudo`). |
| 2 | EC2 reboot results in lost Swap | Forgot the `/etc/fstab` configuration step. Added automount configuration on system boot. |

---

### References

- [Amazon EC2 Getting Started](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
- [How to add swap space to Amazon EC2](https://repost.aws/knowledge-center/ec2-memory-swap-file)
- [Amazon EC2 Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)
