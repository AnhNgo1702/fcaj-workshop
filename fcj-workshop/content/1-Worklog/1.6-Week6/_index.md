---
title: "Worklog Week 6"
date: 2026-06-23
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Objectives

Deploy all application components (Next.js Frontend, Spring Boot Backend, Hermes Agent) into Docker containers and connect them via Internal APIs within the same Docker network.

---

### Completed Tasks

- [x] Dockerize the Spring Boot Backend and run it on Port 8080.
- [x] Dockerize the Next.js Frontend and run it on Port 3000.
- [x] Dockerize and deploy the Hermes Agent on Port 5000.
- [x] Configure internal communication between containers using service names instead of IPs.
- [x] Set up Webhook connection between Hermes Agent and Telegram API.

---

### Implementation Details

**1. Write Dockerfile for each service**
Each service requires its own `Dockerfile`:
- **Backend (Spring Boot)**: Use a JDK image to build the `.jar` file and run the app.
- **Frontend (Next.js)**: Use a Node.js image, run `npm run build`, and start the server.
- **Hermes Agent**: Depending on the language (Python/Nodejs), use the corresponding image.

**2. Integrate into docker-compose.yml**
Extend the `docker-compose.yml` file from Week 5:
```yaml
version: '3.8'
services:
  # ... (mysql-db config)
  backend:
    build: ./backend
    container_name: spring-boot-backend
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=mysql-db
    networks:
      - my-app-network
    depends_on:
      - mysql-db

  frontend:
    build: ./frontend
    container_name: nextjs-frontend
    ports:
      - "3000:3000"
    networks:
      - my-app-network

  hermes-agent:
    build: ./hermes
    container_name: hermes-agent
    ports:
      - "5000:5000"
    networks:
      - my-app-network
```

**3. Communicate via Internal API**
Since the containers share the `my-app-network`, they can call each other via their `container_name`. For example, Spring Boot connects to the database via `mysql-db:3306`, and the Frontend calls the Backend API via `http://spring-boot-backend:8080`.

**4. Launch the entire system**
Run `docker-compose up -d --build` to build images and start all containers simultaneously.

---

### Challenges & Solutions

| # | Challenge | Solution |
|---|-----------|----------|
| 1 | Backend cannot connect to Database upon startup | The database takes a few seconds to start up, while the backend starts too quickly. Configured a retry script or an auto-reconnect mechanism in Spring Boot. |
| 2 | EC2 hangs when building Docker images | The `t3.micro` instance lacks RAM when building Node.js or Java apps. Adding Swap memory (in Week 3) successfully resolved this. |

---

### References

- [Dockerizing a Spring Boot Application](https://spring.io/guides/gs/spring-boot-docker/)
- [Deploying Next.js with Docker](https://nextjs.org/docs/deployment#docker-image)
- [Networking in Compose](https://docs.docker.com/compose/networking/)
