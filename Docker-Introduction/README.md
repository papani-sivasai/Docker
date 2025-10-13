# Docker from Scratch

If you’re new to Docker — welcome!  
This guide walks you through the fundamentals: from understanding what a **Virtual Machine (VM)** is, to grasping **containers**, comparing architectures, and finally installing **Docker** on your local machine and an **AWS VM**.

By the end, you’ll have:
- A clear mental model of how Docker works.
- A working Docker setup on both local and cloud environments.

---

## What is a Virtual Machine (VM)?

A **Virtual Machine (VM)** is like renting an entire house for each app.  
You get privacy and your own kitchen (kernel), but you also pay for a lot of space you might not use.

### Formal Definition

- A VM is a **full computer running a complete OS** on top of a hypervisor.  
- Each VM has its own **virtualized hardware**, **kernel**, and **system drivers**.

### Why They’re Great
- Strong isolation between VM's. One VM crashing won’t affect others.
- Can run **different OS** on the same hardware.
- Ideal for **long-lived, stateful workloads** and **strong security boundaries**.

### Why They’re Heavy
- Booting an entire OS per app is **slow**.
- Larger **memory footprint** and **storage** requirements.
- Scaling many small services leads to **wasted resources**.

---

## What are Containers?

Containers are like separate rooms in the same house.  
Each room can be decorated differently (app + libraries), but all share the same roof (the host kernel).

### Formal Definition
- A **container** is an **isolated process** on the host OS.  
- Isolation uses Linux **namespaces**: PID, NET, MNT, UTS, IPC, USER.  
- **Images** are built in **layers**, making them fast to build, pull, and share.  
- **Docker** is the developer-friendly toolkit for building and running containers  
  (with components like `dockerd`, `containerd`, and `runc` under the hood).

### Why They’re Great
- Start in **milliseconds** and comes with a minimal overhead.  
- Ship the **exact runtime and dependencies** in a repeatable image.  
- Perfect for **microservices**, **CI/CD**, and **scalable systems**.

### Trade-offs
- Containers share the **host kernel** (Linux), making them faster but with a different security boundary.  
- On **macOS** or **Windows**, Docker runs a lightweight **VM behind the scenes** (e.g., Docker Desktop).

---

## Virtual Machines vs Containers

| Feature | Virtual Machines (VMs) | Containers |
|----------|-------------------------|------------|
| **OS** | Each VM includes its own guest OS | Share the host OS kernel |
| **Isolation** | Strong, via hypervisor | Process-level via namespaces |
| **Performance** | Heavy, slow to start | Lightweight, fast startup |
| **Use Case** | Long-lived, secure, multi-OS workloads | Short-lived, Scalable microservices, CI/CD |
| **Analogy** | Renting an entire house | Separate apartments under one roof |

### Analogy
- **VMs** → Each tenant rents an entire house (own kitchen, bathroom, roof).  
- **Containers** → Multiple apartments share one roof (kernel), but each has its own keys and space.

---

## When to Use What

- **Use VMs when:**
  - You need strong isolation or different OS.
  - Running heavy, persistent workloads.

- **Use Containers when:**
  - You need lightweight, scalable, portable apps.
  - Working in microservices, CI/CD pipelines, or cloud-native environments.

---

## Install Docker on AWS (Ubuntu EC2)

Assuming you’ve launched an **Ubuntu EC2 instance** and have SSH access:

```bash
# SSH into the instance
ssh -i your-key.pem ubuntu@ec2-xx-xx-xx-xx.compute-1.amazonaws.com

# Update + prerequisites
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg


# Add Docker’s GPG key and repository
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine and plugins
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker

# Add ubuntu user to docker group
sudo usermod -aG docker $USER

# Verify installation
sudo docker run hello-world

```

## Install Docker on Mac OS

```bash
# Docker Desktop (recommended for most)
brew install --cask docker

```
Once installed, open Docker Desktop from Launchpad and ensure the Docker whale 🐳 icon appears in your menu bar.