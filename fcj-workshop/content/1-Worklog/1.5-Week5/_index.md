---
title: "Worklog Week 5"
date: 2026-06-23
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Objectives

Set up a containerization environment on EC2 using Docker and Docker Compose, create an internal network (bridge network), and deploy the MySQL database running independently inside a container.

---

### Completed Tasks

- [x] Install Docker and Docker Compose on Amazon Linux 2023 / Ubuntu.
- [x] Configure permissions so the user can run docker commands without `sudo`.
- [x] Create a `bridge` Docker network so containers can communicate using local domain names.
- [x] Deploy the MySQL container via a `docker-compose.yml` file.
- [x] Connect and verify the database from inside the EC2 instance.

---

### Implementation Details

**1. Install Docker & Docker Compose**
- Run commands to update and install the Docker engine:
```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable docker
sudo systemctl start docker
# Grant permissions to the default user (ubuntu)
sudo usermod -aG docker ubuntu
```
Log out and log back in for the permissions to take effect.

**2. Initialize Docker Network**
- Create a private network (bridge) so MySQL, Spring Boot, and Next.js can communicate securely without being unnecessarily exposed:
```bash
docker network create my-app-network
```

**3. Deploy MySQL Container**
- Create a data directory and the `docker-compose.yml` file:
```yaml
version: '3.8'
services:
  mysql-db:
    image: mysql:8.0
    container_name: mysql-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: my-secret-pw
      MYSQL_DATABASE: my_app_db
    ports:
      - "3306:3306"
    networks:
      - my-app-network
    volumes:
      - db_data:/var/lib/mysql

networks:
  my-app-network:
    external: true

volumes:
  db_data:
```
- Launch the container in detached mode: `docker-compose up -d`.

---

### Challenges & Solutions

| # | Challenge | Solution |
|---|-----------|----------|
| 1 | `permission denied` error when running `docker ps` | This is because I hadn't logged out and logged back in after adding the user to the `docker` group. |
| 2 | MySQL container stops immediately after starting | Usually caused by a missing mandatory environment variable (like `MYSQL_ROOT_PASSWORD`). Added it to the compose file and re-ran. |

---

### References

- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Docker Compose Overview](https://docs.docker.com/compose/)
- [Docker Hub - MySQL Official Image](https://hub.docker.com/_/mysql)
