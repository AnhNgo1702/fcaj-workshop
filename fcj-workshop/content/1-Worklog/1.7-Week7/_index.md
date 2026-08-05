---
title: "Worklog Week 7"
date: 2026-06-23
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Objectives

Install Nginx as a Reverse Proxy to route external traffic to the appropriate Docker containers, configure DNS with Amazon Route 53, and set up HTTPS.

---

### Completed Tasks

- [x] Install Nginx on EC2 (directly on the Host, not in Docker).
- [x] Configure Nginx Reverse Proxy to route requests (`/api/*` to Backend, `/` to Frontend).
- [x] Register a domain and configure a Hosted Zone on Amazon Route 53.
- [x] Create a DNS Record pointing the domain to the EC2's public IP.
- [x] Register and install a free SSL/TLS certificate via Let's Encrypt (Certbot) for Nginx.

---

### Implementation Details

**1. Install and Configure Nginx**
- Install Nginx on EC2:
```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```
- Create the configuration file `/etc/nginx/conf.d/myapp.conf`:
```nginx
server {
    listen 80;
    server_name mydomain.com;

    # Route to Frontend
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Route to Backend API
    location /api/ {
        proxy_pass http://localhost:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
- Restart Nginx: `sudo systemctl restart nginx`.

**2. Configure Amazon Route 53**
- In the Route 53 console, create a Hosted Zone for your domain.
- Create an **A** Record, pasting the EC2 instance's Public IPv4 address into the Value field.
- Wait for DNS propagation.

**3. Set up HTTPS with Certbot**
To protect data (especially for API calls and Telegram webhooks), install SSL:
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d mydomain.com
```
Certbot will automatically modify the Nginx configuration file to add SSL settings on port 443 and set up an auto-renewal script.

---

### Challenges & Solutions

| # | Challenge | Solution |
|---|-----------|----------|
| 1 | Accessing domain returns 502 Bad Gateway | Checked Nginx logs (`/var/log/nginx/error.log`). The cause was the Backend container not running. Re-ran `docker-compose up -d`. |
| 2 | Telegram Webhook not working | Telegram requires webhooks to use HTTPS. After installing Certbot SSL in step 3, the webhook functioned normally. |

---

### References

- [NGINX Reverse Proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
- [Amazon Route 53 Getting Started](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/getting-started.html)
- [Certbot Instructions for Nginx](https://certbot.eff.org/instructions)
