# Docker Compose — Simplifying Multi-Container Apps

> Learn how to orchestrate multi-container applications using Docker Compose.

In the previous article, we manually connected Flask and MySQL containers using Docker networks. That worked but typing long `docker run` commands quickly becomes tedious.

With **Docker Compose**, we can define everything like images, networks, volumes, and environment variables in a single YAML file and spin it all up with **one command**:

```bash
docker compose up -d
```

---

## What You’ll Learn

* What Docker Compose is and why it matters
* Step-by-step process to write a `docker-compose.yml`
* How to use `.env` files for cleaner environment management
* How to run and manage Flask + MySQL containers together
* Common Compose commands you’ll use every day

---

## App Overview

We’ll use a simple **Flask To-Do App** with **MySQL** as the database.

GitHub Repository: [two-tier--flask-app](https://github.com/papani-sivasai/two-tier--flask-app)

```bash
git clone https://github.com/papani-sivasai/two-tier--flask-app
cd two-tier--flask-app
```

---

## Step 1: The Dockerfile (Flask App)

```dockerfile
# Start from a lightweight Python base image
FROM python:3.11-slim

# Set working directory inside container
WORKDIR /app

# Copy dependency list and install them
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy all project files
COPY . .

# Run Flask server
CMD ["flask", "run", "--host=0.0.0.0", "--port=5000"]
```

This builds a Flask image and exposes port **5000**.

---

## Step 2: Why Use Docker Compose?

Without Compose:

```bash
docker build -t flask-todo .
docker run -d --name flask-todo --network flask-network --env-file ./env flask-todo
```

You have to remember **every flag, port, and environment variable.**

With Compose:

```bash
docker compose up -d
```

That’s it! Docker Compose:

* Creates both containers (Flask + MySQL)
* Links them via a network
* Sets up volumes
* Injects environment variables
* Runs everything in order

---

## Step 3: Define Environment Variables

Create a `.env` file to manage sensitive or reusable variables.

```ini
DB_HOST=db
DB_USER=root
DB_NAME=todo_db
DB_PASSWORD=my-secret-pw
DB_PORT=3306
FLASK_PORT=5000
MYSQL_ROOT_PASSWORD=my-secret-pw
MYSQL_DATABASE=todo_db
```

> Compose automatically loads `.env` from the same directory as your `docker-compose.yml`.

---

## Step 4: Writing the `docker-compose.yml`

We’ll define two services — `db` (MySQL) and `web` (Flask).

### Add MySQL Service

```yaml
services:
  db:
    image: mysql:8.0
    container_name: todo-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
    volumes:
      - mysql_data:/var/lib/mysql
      - ./db-init:/docker-entrypoint-initdb.d:ro
    networks:
      - todo-network
```

**Explanation:**

* `image`: use the official MySQL image
* `restart`: auto-restart on failure
* `environment`: inject DB credentials from `.env`
* `volumes`: persist data (`mysql_data`) + init SQL scripts (`db-init`)
* `networks`: attaches to `todo-network`

---

### Add Flask Service

```yaml
  web:
    build: .
    container_name: todo-flask
    restart: always
    ports:
      - "${FLASK_PORT}:5000"
    env_file:
      - .env
    depends_on:
      - db
    networks:
      - todo-network
```

**Explanation:**

* `build`: build image from current folder’s Dockerfile
* `ports`: map Flask’s 5000 port to host `${FLASK_PORT}`
* `env_file`: loads `.env` automatically
* `depends_on`: ensures MySQL starts first
* `networks`: joins the same network as MySQL

---

### Define Networks and Volumes

```yaml
networks:
  todo-network:

volumes:
  mysql_data:
```

**This ensures:**

* `todo-network` is auto-created by Compose
* `mysql_data` persists your DB data

---

### Final `docker-compose.yml`

```yaml
services:
  db:
    image: mysql:8.0
    container_name: todo-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
    volumes:
      - mysql_data:/var/lib/mysql
      - ./db-init:/docker-entrypoint-initdb.d:ro
    networks:
      - todo-network

  web:
    build: .
    container_name: todo-flask
    restart: always
    ports:
      - "${FLASK_PORT}:5000"
    env_file:
      - .env
    depends_on:
      - db
    networks:
      - todo-network

networks:
  todo-network:

volumes:
  mysql_data:
```

---

## Step 5: Run Everything

```bash
docker compose up -d
```

Expected output:

```
✔ Network todo-network  Created
✔ Volume mysql_data     Created
✔ Container todo-mysql  Started
✔ Container todo-flask  Started
```

Visit [http://localhost:5000](http://localhost:5000) — your Flask app is live and connected to MySQL.

---

## Step 6: How Compose Uses `.env`

Docker Compose automatically loads environment variables from `.env` in the same directory.

Confirm by running:

```bash
docker compose config
```

This shows the final merged configuration with all variables resolved.

---

## Step 7: Useful Compose Commands

| Command                   | Description                   |
| ------------------------- | ----------------------------- |
| `docker compose up -d`    | Start all services            |
| `docker compose down`     | Stop and remove containers    |
| `docker compose ps`       | List running containers       |
| `docker compose logs web` | View Flask logs               |
| `docker compose build`    | Rebuild images                |
| `docker compose down -v`  | Remove containers and volumes |

---

## Troubleshooting Tips

| Issue                     | Likely Cause     | Fix                                                        |
| ------------------------- | ---------------- | ---------------------------------------------------------- |
| Flask can’t connect to DB | Wrong `DB_HOST`  | Use `db` (Compose service name)                            |
| Data lost after restart   | No named volume  | Ensure `mysql_data` is defined                             |
| Port conflict             | Host port in use | Change mapping (e.g., `5001:5000`)                         |
| `.env` not applied        | Wrong path       | Ensure `.env` is in same directory as `docker-compose.yml` |

---

## Summary

You’ve now built a **fully containerized multi-service stack** (Flask + MySQL) managed entirely by **Docker Compose**.

With a single file, Compose handled:

* Image builds
* Networking
* Persistent storage
* Environment management
* Startup order

And your app is up and running with one command:

```bash
docker compose up -d
```

---

## Next Up

**In next part we will Containerise a Three-Tier E-Commerce Application**
We’ll combine everything we’ve learned so far. Images, containers, volumes, networks, and Compose to deploy a realistic multi-tier application.

---

**Author:** *Siva Sai Papani*
**Article Source:** [Docker Compose on Hashnode](https://docker-part6.hashnode.dev/docker-compose-simplifying-multi-container-apps)
