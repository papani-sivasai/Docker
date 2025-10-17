# Docker Networking — Connecting Containers

> Learn how to make multiple containers communicate seamlessly using Docker networks.

In the previous article, we learned how to persist data using Docker Volumes. Now it’s time to take our setup one step further by making multiple containers talk to each other.

We’ll use a **two-tier Flask To-Do App with MySQL** as our example to understand Docker Networking in action.

GitHub Repo: [two-tier--flask-app](https://github.com/papani-sivasai/two-tier--flask-app)

---

## What You’ll Learn

* What Docker networks are and why they matter
* Types of Docker networks (**bridge**, **host**, **none**, **overlay**)
* How to connect Flask to MySQL using a **user-defined bridge network**
* Step-by-step setup using real Docker commands
* Common networking commands, troubleshooting, and visual diagrams

---

## Why Docker Networking?

By default, Docker containers are **isolated**,  they don’t know about each other. If you start two containers separately (like Flask and MySQL), Flask cannot connect to MySQL using `localhost` because each container runs in its own environment.

**Without a network:**

```
Flask ➞ cannot find MySQL (connection refused)
```

**With a network:**

```
Flask ➞ communicates with MySQL using its container name (todo-mysql)
```

### Docker networks solve this by providing:

* Internal **DNS name resolution** (containers can find each other by name)
* **Isolation** from unrelated containers
* A secure, **virtual network layer**

---

## Types of Docker Networks

| Network Type | Description                                                                                                       | Use Case                                        |
| ------------ | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| **bridge**   | Default for standalone containers. Each gets a private IP and can talk to others on the same network using names. | Local development, two-tier apps, microservices |
| **host**     | Shares host’s network stack (no isolation). Uses host ports directly.                                             | High-performance apps needing full host access  |
| **none**     | Completely isolated — no network.                                                                                 | Security or offline jobs                        |
| **overlay**  | Connects containers across multiple Docker hosts.                                                                 | Multi-host / Swarm setups                       |

**Key takeaway:** For 90% of local setups, use a **user-defined bridge network**. It provides isolation and name-based communication.

---

## How Docker Networking Works

Think of a Docker network as a **private LAN** inside your computer.

Each container gets:

* Its own **IP address**
* A **hostname** (same as container name)
* Access to other containers on the same network

When Flask connects to MySQL using `todo-mysql`, Docker’s DNS automatically resolves it to that container’s IP.

---

## Hands-On: Flask + MySQL To-Do App

We’ll use the [two-tier-flask-app](https://github.com/papani-sivasai/two-tier--flask-app) repo.

### App structure:

* **Flask API container** → `two-tier-todo`
* **MySQL container** → `todo-mysql`
* Shared network → `todo-network`

---

### Step 1: Clone the Repository

```bash
git clone https://github.com/papani-sivasai/two-tier--flask-app.git
cd two-tier--flask-app
```

Inside, you’ll find:

* `app/` → Flask backend source code
* `Dockerfile` → Flask image definition
* `.env` → environment variables (DB credentials, host, port)
* `db-init/` → SQL init scripts for MySQL

---

### Step 2: Create a Custom Docker Network

```bash
docker network create todo-network
```

Verify:

```bash
docker network ls
```

Output example:

```bash
NETWORK ID     NAME           DRIVER    SCOPE
032460e354c0   todo-network   bridge    local
```

---

### Step 3: Run the MySQL Container

```bash
docker run -d \
  --name todo-mysql \
  --network todo-network \
  -e MYSQL_ROOT_PASSWORD=my-secret-pw \
  -e MYSQL_DATABASE=todo_db \
  -p 3306:3306 \
  -v "$PWD/db-init":/docker-entrypoint-initdb.d:ro \
  mysql:8.0
```

**Explanation:**

* `--network todo-network` → joins MySQL to the custom network.
* `-e` → sets root password and database.
* `-p 3306:3306` → exposes DB to host.
* `-v "$PWD/db-init":/docker-entrypoint-initdb.d:ro` → runs init scripts.

Check logs:

```bash
docker logs -f todo-mysql
```

Wait until you see **"ready for connections"**.

---

### Step 4: Run the Flask Container

Build the image:

```bash
docker build -t two-tier-app .
```

Run Flask:

```bash
docker run -d \
  --name two-tier-todo \
  --network todo-network \
  -p 5000:5000 \
  --env-file .env \
  two-tier-app
```

**Explanation:**

* `--network todo-network` → connects Flask to MySQL network.
* `-p 5000:5000` → maps port.
* `--env-file .env` → loads DB credentials and host (like `DB_HOST=todo-mysql`).

---

### Step 5: Verify Connectivity

```bash
docker logs two-tier-todo
```

Expected output:

```
Connected to MySQL at host todo-mysql:3306
Running on http://0.0.0.0:5000
```

Visit [http://localhost:5000](http://localhost:5000) — your app should be live!

---

## Inspect the Network

```bash
docker network inspect todo-network
```

Example output snippet:

```json
"Containers": {
  "todo-mysql": {
    "Name": "todo-mysql",
    "IPv4Address": "172.18.0.2/16"
  },
  "two-tier-todo": {
    "Name": "two-tier-todo",
    "IPv4Address": "172.18.0.3/16"
  }
}
```

Both containers have private IPs within the same subnet and can communicate via names.

---

## Network Isolation in Action

Run a random container **not** on the network:

```bash
docker run -it alpine ping todo-mysql
```

Result:

```
ping: bad address 'todo-mysql'
```

Only containers on the same network can resolve each other — this provides secure isolation.

---

## Common Docker Networking Commands

| Command                                       | Description                  |
| --------------------------------------------- | ---------------------------- |
| `docker network ls`                           | List all networks            |
| `docker network create <name>`                | Create a new network         |
| `docker network inspect <name>`               | View network details         |
| `docker network connect <net> <container>`    | Connect container to network |
| `docker network disconnect <net> <container>` | Disconnect container         |
| `docker network rm <name>`                    | Remove a custom network      |

---

## Troubleshooting Tips

* **Flask can’t connect to DB?**
  → Ensure both containers are on `todo-network`.

* **MySQL not ready?**
  → Wait for `ready for connections` in `docker logs todo-mysql`.

* **DB lost after restart?**
  → Check your volume mounting.

---

## Summary

* Containers are isolated by default — networks connect them.
* Use **user-defined bridge networks** for apps that talk to each other.
* Containers on the same network can communicate by **name**, not IP.
* Manage networks easily with `docker network` commands.

---

## ✨ Coming Up Next: Docker Compose

Manually running multiple containers is fine for demos but not scalable.
In the next article, we’ll simplify everything using **Docker Compose**:

* Define Flask and MySQL in one YAML file.
* Automatically create networks and volumes.
* Start your entire stack with **`docker compose up -d`**.

---

**Author:** *Siva Sai Papani*
**Article Source:** [Docker Networking on Hashnode](https://docker-part5.hashnode.dev/docker-networking-connecting-containers)
