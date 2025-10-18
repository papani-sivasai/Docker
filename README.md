# Docker — Hands-On Projects Repository
Master the docker with hands-on project for each topic

> Master Docker by doing, this repo contains hands-on projects covering Docker fundamentals, images, containers, networks, volumes, compose, and sample multi-tier apps.

---

## About

This repository is designed as a **practical learning resource for Docker**. Instead of just reading theory, you’ll **work through exercises and real-world sample applications** for core Docker concepts, including:

* Images & containers
* Dockerfile and building images
* Volumes & persistent data
* Networking between containers
* Multi-container / multi-tier setups
* Compose orchestration
* and more!

Each topic has its own folder with example code and instructions.

---

## Contents

Here’s a sample layout:

```bash
├── Docker-Introduction
├── Docker-Images
├── Docker-Containers
├── Docker-Volumes
├── Docker-Networks
├── Docker-Compose
├── three-tier-project
└── README.md
```

**Folder Overview:**

| Folder                | Description                                                 |
| --------------------- | ----------------------------------------------------------- |
| `Docker-Introduction` | Basic concepts, installation, first “hello world” container |
| `Docker-Images`       | Building, tagging, and pushing images                       |
| `Docker-Containers`   | Running, managing, and container lifecycle                  |
| `Docker-Volumes`      | Data persistence across containers                          |
| `Docker-Networks`     | Container communication and networking                      |
| `Docker-Compose`      | Define and orchestrate multi-container apps                 |
| `three-tier-project`  | Full-stack sample app (frontend + backend + database)       |

---

## Prerequisites

Before you begin, ensure you have the following installed:

*  **Docker Desktop**
* **Docker Compose** (if not bundled)

Basic familiarity with the command line and web app concepts will help when exploring examples.

---

## Getting Started

Clone this repository:

```bash
git clone https://github.com/papani-sivasai/Docker.git
cd Docker
```

Pick a topic folder you want to explore, for example:

```bash
cd Docker-Compose
```

Then follow the instructions in that folder’s README usually including commands like building images, running containers, or connecting services.

**Experiment** Modify Dockerfiles, change ports, attach volumes, or add new services.

---

## Project Examples

| Topic                  | What You’ll Do                                  | Commands / Notes                                |
| ---------------------- | ----------------------------------------------- | ----------------------------------------------- |
| **Docker-Images**      | Build and tag a custom image                    | `docker build -t myapp:latest .`                |
| **Docker-Containers**  | Run containers and manage lifecycle             | `docker run ...`, `docker ps`, `docker rm`      |
| **Docker-Volumes**     | Persist data across restarts                    | `docker run -v host_path:container_path ...`    |
| **Docker-Networks**    | Link multiple containers                        | `docker network create`, `docker run --network` |
| **Docker-Compose**     | Define services in YAML                         | `docker compose up --build`                     |
| **three-tier-project** | Launch full-stack app (frontend + backend + DB) | `docker compose up`                             |

---


## License

This repository is **open source**. You may use and modify it for personal or educational purposes.

---

**Author:** *Siva Sai Papani*
