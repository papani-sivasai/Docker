# Docker Images From Zero to Hero

> Master Docker with hands-on projects for each topic.

This guide covers **everything you need to know** about Docker images, how they work, how to build them, and how to push them to Docker Hub.

---

## What You’ll Learn

* What Docker images are and how they work
* How to write a **Dockerfile** (with clear explanations)
* How to **build**, **run**, **pull**, and **push** images
* Hands-on recap
* A list of useful Docker image commands

---

## What is a Docker Image?

A **Docker image** is like a **blueprint** for your application. It includes everything your app needs to run: **code**, **libraries**, **dependencies**, and **configurations**.

**Analogy:**

* **Image → Recipe** (instructions to cook a dish)
* **Container → The dish itself** (running instance of the image)

Without an image, you can’t create a container.

---

## Layers in a Docker Image

Each instruction in a Dockerfile (`FROM`, `RUN`, `COPY`, etc.) creates a new **layer**. These layers are stacked to form the final image, and they are **cached** to make builds faster.

To see the layers of an image:

```bash
docker history <image-name>
```

---

## Building Docker Images

To build an image, we need a **Dockerfile**, Dockerfile is a simple text file with a set of build instructions.

We’ll use the **Python Flask To-Do app** for this demo:
[docker-python-flask-todo-app](https://github.com/papani-sivasai/docker-python-flask-todo-app)

---

### Step 1: Write a Dockerfile

```dockerfile
# Start from a lightweight Python base image
FROM python:3.11-slim

# Set a working directory inside the container
WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy the rest of the application code to the working directory
COPY . .

# Set environment variables
ENV FLASK_APP=app.py
ENV FLASK_ENV=production

# Expose the port Flask runs on
EXPOSE 5000

# Run Flask when the container starts
CMD ["flask", "run", "--host=0.0.0.0", "--port=5000"]
```

---

### Dockerfile Explained (Line by Line)

| Line                                                              | Explanation                                                            |
| ----------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `FROM python:3.11-slim`                                           | Chooses Python 3.11 slim image as the base. "Slim" = smaller & faster. |
| `WORKDIR /app`                                                    | Sets `/app` as the working directory inside the container.             |
| `COPY requirements.txt .` + `RUN pip install -r requirements.txt` | Copies dependencies and installs them inside the container.            |
| `COPY . .`                                                        | Copies all your project files (code, templates, static files).         |
| `ENV FLASK_APP=app.py` + `ENV FLASK_ENV=production`               | Defines environment variables inside the container.                    |
| `EXPOSE 5000`                                                     | Documents that the app will use port 5000.                             |
| `CMD ["flask", "run", "--host=0.0.0.0", "--port=5000"]`           | Runs the Flask app when the container starts.                          |

---

### Step 2: Build the Image

```bash
docker build -t flask-todo-app:1.0 .
```

* `-t flask-todo-app:1.0` → tags the image with a name and version.
* `.` → uses the current directory as the build context.

Check if the image was created:

```bash
docker images
```

---

### Step 3: Run a Container

```bash
docker run -d -p 5000:5000 --name flask-todo flask-todo-app:1.0
```

**Flags explained:**

| Flag                | Description                                    |
| ------------------- | ---------------------------------------------- |
| `-d`                | Run in detached mode (background).             |
| `-p 5000:5000`      | Maps local port 5000 to container’s port 5000. |
| `--name flask-todo` | Names the container “flask-todo”.              |

Check running containers:

```bash
docker ps
```

View logs:

```bash
docker logs flask-todo
```

Stop the container:

```bash
docker stop flask-todo
```

Remove the container:

```bash
docker rm flask-todo
```

---

## Pulling Docker Images

You don’t always need to build from scratch, you can pull existing images from Docker Hub:

```bash
docker pull nginx:alpine
```

If no tag is specified, Docker defaults to **latest**.

List all images:

```bash
docker images
```

---

## Pushing Docker Images

To share your image:

### Step 1: Login

```bash
docker login
```

### Step 2: Tag the Image

```bash
docker tag flask-todo-app:1.0 <your-dockerhub-username>/flask-todo-app:1.0
```

### Step 3: Push the Image

```bash
docker push <your-dockerhub-username>/flask-todo-app:1.0
```

Now anyone can pull it:

```bash
docker pull <your-dockerhub-username>/flask-todo-app:1.0
```

---

## Hands-On Recap

```bash
# Clone repo
git clone https://github.com/papani-sivasai/docker-python-flask-todo-app
cd docker-python-flask-todo-app

# Build image
docker build -t flask-todo-app:1.0 .

# Run container
docker run -d -p 5000:5000 flask-todo-app:1.0

# Tag & push to Docker Hub
docker tag flask-todo-app:1.0 <your-dockerhub-username>/flask-todo-app:1.0
docker push <your-dockerhub-username>/flask-todo-app:1.0

# Pull & run anywhere
docker pull <your-dockerhub-username>/flask-todo-app:1.0
```

---

## Docker Image Commands (Cheat Sheet)

```bash
# Build an image
docker build -t name:tag .

# List all images
docker images

# Remove an image
docker rmi image_id

# Pull an image
docker pull image_name:tag

# Push an image
docker push username/image_name:tag

# View image layers
docker history image_name:tag

# Inspect detailed info
docker inspect image_name:tag
```

---

## Summary

* Docker **images** are blueprints for **containers**.
* A **Dockerfile** defines how to build an image.
* Use:

  * `docker build` → to create an image
  * `docker run` → to start a container
  * `docker pull` / `docker push` → to share images

---

## Next Up

In the next guide, we’ll explore **Docker Containers** in depth using a **BMI Calculator React App**.

---

**Author:** *[Siva Sai Papani]*
**Article Source:** [Docker Images on Hashnode](https://docker-part2.hashnode.dev/docker-images-beginner-friendly-guide)
