# 🐳 Dockerized Todo App with MySQL Backend

Welcome to the **Dockerized Todo App** guide! This guide walks you through containerizing a Node.js application, adding a MySQL database, and using Docker Compose for managing a multi-container environment.

---

![Docker Compose Overview](https://goldeneagle.ai/media/media/uploads/docker.gif)

---

## 🔑 Key Topics Covered

- Containerizing a Node.js application
- Building Docker images
- Running single containers
- Connecting containers via Docker networks
- Adding a MySQL backend
- Using Docker Compose for multi-container orchestration

---

## 📦 Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- Text editor (e.g., [VS Code](https://code.visualstudio.com/))

---

## 🚀 Getting Started

### Clone the Repository
```bash
git clone https://github.com/docker/getting-started-app.git
cd getting-started-app
```

### Dockerfile
Create a `Dockerfile` in the project root:
```Dockerfile
# syntax=docker/dockerfile:1
FROM node:lts-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```

### Build Docker Image
```bash
docker build -t getting-started .
```

### Run Application in a Container
```bash
docker run -d -p 127.0.0.1:3000:3000 getting-started
```
Visit [http://localhost:3000](http://localhost:3000) to view the app.

---

## 🔁 Update and Rebuild

Update the empty todo message in `src/static/js/app.js`:
```diff
- <p className="text-center">No items yet! Add one above!</p>
+ <p className="text-center">You have no todo items yet! Add one above!</p>
```

Rebuild and run:
```bash
docker build -t getting-started .
docker stop <container-id>
docker rm <container-id>
docker run -dp 127.0.0.1:3000:3000 getting-started
```

---

## 🧱 Add MySQL Backend

### Create Docker Network
```bash
docker network create todo-app
```

### Run MySQL Container
```bash
docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:8.0
```

### Verify MySQL Setup
```bash
docker exec -it <mysql-container-id> mysql -u root -p
# Password: secret
mysql> SHOW DATABASES;
```

---

## 🔌 Connect Node App to MySQL

Run your app with MySQL environment variables:
```bash
docker run -dp 127.0.0.1:3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:18-alpine \
  sh -c "yarn install && yarn run dev"
```

Check logs:
```bash
docker logs -f <container-id>
```

---

## 🧩 Docker Compose Setup

### `compose.yaml`
```yaml
services:
  app:
    image: node:18-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 127.0.0.1:3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:8.0
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

### Launch with Compose
```bash
docker compose up -d
```

### View Logs
```bash
docker compose logs -f
```

---

## 🧹 Tear Down Stack
```bash
docker compose down --volumes
```

---

## 📚 Helpful Links
- [Docker Getting Started Guide](https://docs.docker.com/get-started/)
- [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [MySQL Docker Hub](https://hub.docker.com/_/mysql)
- [Yarn Docs](https://classic.yarnpkg.com/en/docs/cli/)

---

Enjoy building with Docker! 🚢
