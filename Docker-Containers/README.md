# Docker Containers — A Practical Guide

This guide walks you through Docker containers, what they are, how they differ from images, how to run them interactively or in the background, and how to manage their lifecycle.

---

## What You’ll Learn

* What Docker containers are (and how they differ from images)
* Running containers in **interactive** vs **detached** mode
* Managing container lifecycle (**start**, **stop**, **logs**, **exec**, **remove**)
* Hands-on example with the **BMI Calculator React app**
* A cheat sheet of essential Docker container commands

---

## What is a Docker Container?

A **Docker container** is a **running instance of a Docker image**.

**Analogy:**

* **Image = Recipe** (instructions)
* **Container = Cooked Dish** (running instance)

### Containers are:

* Lightweight and isolated
* Based on an image but have their own writable layer
* Disposable: you can start/stop/remove them anytime
* Able to run multiple instances with different names from the same image

---

## Running Containers

### Basic Run Command

```bash
docker run image-name
```

Creates and starts a container from the image.

---

### Interactive Mode (`-it`)

```bash
docker run -it ubuntu
```

* `-i` → interactive (keeps STDIN open)
* `-t` → allocates a terminal

**When to use:**

* When you want to interact directly with the container
* Testing commands in Linux images like `ubuntu` or `nginx`
* Debugging by checking logs, config, or filesystem inside a container

**Example:**

```bash
docker run -it ubuntu
```

Now you are inside a running Ubuntu container shell. You can run commands like `ls`, `apt-get update`, etc.

---

### Detached Mode (`-d`)

```bash
docker run -d nginx
```

* `-d` → detached mode (runs in the background)

**When to use:**

* Running web servers, apps, or services that should stay running in the background (React, Flask, Nginx, etc.)

---

### When to Use `-d` vs `-it`

| Mode  | Use Case                                               |
| ----- | ------------------------------------------------------ |
| `-it` | For interactive tasks like debugging or manual testing |
| `-d`  | For long-running services like web apps or APIs        |

---

### Naming Containers

```bash
docker run -d --name myapp bmi-calculator-app
```

* `--name` → gives your container a friendly and readable name.

---

## Managing Containers

### List Running Containers

```bash
docker ps
```

### List All Containers (including stopped)

```bash
docker ps -a
```

### View Logs

```bash
docker logs container-name
```

### Stop and Restart Containers

```bash
docker stop container-name
docker start container-name
```

### Remove Containers

```bash
docker rm container-name
```

---

### Access a Running Container with `exec`

```bash
docker exec -it container-name sh
```

This opens a shell inside a running container.

**When to use:**

* To inspect logs/config inside a live app
* To debug issues without stopping the container
* To run one-off commands inside the app environment

**Example:**

```bash
docker exec -it bmi-app sh
```

Now you’re inside the **BMI Calculator React** container. Run `ls, node_modules` or check processes with `ps aux`.

---

## Hands-on Example (BMI Calculator React App)

We’ll use your **React + Vite BMI Calculator App** as a demo.
Repo: [docker-bmi-calculator-reactJS](https://github.com/papani-sivasai/docker-bmi-calculator-reactJS)

### Step 1: Write a Dockerfile

Create a `Dockerfile` in the project root:

```dockerfile
# Use Node.js base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json package-lock.json ./
RUN npm install --production

# Copy project files
COPY . .

# Expose Vite default port
EXPOSE 5173

# Start the app
CMD ["npm", "run", "dev"]
```

---

### Step 2: Build the Image

```bash
docker build -t bmi-calculator-app .
```

---

### Step 3: Run the Container

```bash
docker run -d -p 5173:5173 --name bmi-app bmi-calculator-app
```

Open **[http://localhost:5173](http://localhost:5173)** — your BMI Calculator is running inside a container!

---

### Step 4: Manage the Container

List running containers:

```bash
docker ps
```

Check logs:

```bash
docker logs bmi-app
```

Access interactively:

```bash
docker exec -it bmi-app sh
```

Stop and restart:

```bash
docker stop bmi-app
docker start bmi-app
```

Remove:

```bash
docker rm bmi-app
```

---

## Docker Container Commands (Cheat Sheet)

```bash
# Run container
docker run [options] image

# Run in interactive mode
docker run -it ubuntu

# Run in detached mode
docker run -d image

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# View container logs
docker logs container-name

# Access a container shell
docker exec -it container-name sh

# Stop a container
docker stop container-name

# Start a container
docker start container-name

# Remove a container
docker rm container-name
```

---

## Summary

* **Images** are blueprints; **containers** are the running apps.
* Learned difference between `-d` (detached) and `-it` (interactive).
* `exec` lets you inspect and debug inside running containers.
* You now know how to **start**, **stop**, **restart**, and **remove** containers.

---

## Next Up

In the next article, we’ll explore **data persistence** using **Docker volumes and bind mounts**.

---

**Author:** *Siva Sai Papani*
**Article Source:** [Docker Containers on Hashnode](https://docker-part3.hashnode.dev/docker-containers-running-and-managing-lifecycle)
